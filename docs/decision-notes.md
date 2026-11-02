# PCB Validator — Decision notes

The choices behind the PCB Validator spoke, the trade-offs I accepted, and what I deliberately left out. This is the short version of the architecture and the mockups.

## Summary

### Server decides. Browser renders. Trace owns the order.

The validator is not an editor and not a second source of truth. It interprets a package, runs profile-specific checks, and returns findings and priceable items.

The customer fixes the design in their EDA tool and uploads a new revision. The spoke earns trust by being deterministic and traceable, not by hiding the work in the browser.

Technology stack:

- Preact + TypeScript
- FastAPI
- Async Python worker
- PostgreSQL
- Object storage

## 01. Architecture choice

Decision: A browser-based viewer, a FastAPI spoke API, and an asynchronous Python validation worker.

Parsing and DFM rules need one authoritative implementation. If they run in the browser, results depend on the client, the device, and how much geometry it can hold in memory.

The server keeps that deterministic, and the worker isolates the slow, stateful part so the API can still serve order context and progress.

The browser only renders normalized geometry, which keeps layer toggling and finding focus immediate.

Trade-off: More moving parts and an async progress path.

I mitigate that with persisted run state, SSE for progress and completion, retries on the worker, and immutable runs that can be reopened or re-run.

## 02. Processing approach

Decision: Parse Gerber and Excellon, normalize geometry, run profile-specific DFM checks, and return findings.

Gerber and Excellon are terse and stateful. A wrong aperture, polarity, or drill unit changes the geometry silently, so there has to be one parser and one normalized representation in board units.

Rules then evaluate that geometry against the manufacturing profile assigned to the order, not against a permissive default.

A finding carries:

- Rule ID
- Measured value
- Required threshold
- Unit
- Layer
- Geometry reference

Trade-off: Parser fidelity is the real risk, not the UI.

I would build against one real customer package first and verify that every highlighted finding lands on the right trace before expanding the rule set.

## 03. Ownership

Decision: Trace owns orders, original files, identity, assigned manufacturing profiles, and checkout. The spoke owns validation runs, findings, rules, and derived artifacts.

Original files and the money path stay in the hub. Interpretation and engineering truth stay in the spoke.

Findings are authored by the spoke but scoped to the order, so checkout can block on them.

The worker fetches files with its own authorized Hub API access; it does not download through the API and it does not talk to PostgreSQL.

It reports metadata and findings to the API, and the API persists run state.

Trade-off: Some state is visible in two places: the hub scopes findings and items, the spoke keeps provenance.

The rule is that the hub is authoritative for the order and the money, and the spoke is authoritative for the run and the geometry.

## 04. Finding resolution

Decision: Customers fix designs in their EDA software, upload a new revision, and rerun validation. Acknowledgement is not resolution.

A validator that can edit geometry becomes a second source of truth and a much larger product.

The honest loop is:

1. Understand the constraint.
2. Change the board in the tool that owns the design.
3. Upload a revision.
4. Let a new run prove the fix.

Acknowledgement, will address, and request manufacturing review are tracking states.

A finding closes only when a later run confirms the geometry is gone.

Trade-off: The loop is slower than a one-click fix, but it is trustworthy.

A finding cannot disappear because someone checked a box, and the order keeps the evidence.

## 05. Data and lifecycle

Decision: Revision-scoped validation runs, persisted state and findings, and object storage for larger derived artifacts.

Every revision is immutable and identified by a content hash.

Every run records:

- Revision hash
- Parser versions
- Ruleset version
- Profile snapshot
- Worker image
- Timestamps
- Outcome

Findings link back to the run, the rule, and the geometry.

A re-run creates a new run, never an in-place edit, so "why did this pass last week" stays answerable.

PostgreSQL holds run state and findings; object storage holds normalized geometry and larger artifacts, and the API reads it when serving the browser.

Trade-off: Storage grows with every revision and run.

The mitigation is content-hash reuse, a retention policy for superseded artifacts, and keeping the source package in Trace rather than copying it into the spoke.

## 06. MVP and non-goals

Decision: Ship the core loop: the supported file formats, layer viewing, a focused set of DFM checks, findings, and order-item handoff.

The MVP is:

- One package shape (RS-274X Gerber plus Excellon).
- Initial support for a defined four-layer board configuration.
- Explicit layer mapping.
- Normalized geometry.
- A board viewer.
- A small rule set.
- Revision upload and revalidation.
- Findings.
- Priceable items posted back to Trace.

The checks I would start with are:

- Clearance.
- Annular ring.
- Drill aspect ratio.
- Soldermask sliver.
- Copper to board edge.

That is enough to prove the loop and enough to be wrong in useful ways.

### In the MVP

- Gerber and Excellon package
- Defined four-layer configuration
- Layer mapping and viewer
- Focused DFM rules
- Findings with geometry
- Revision upload and rerun
- Findings and order items in Trace

### Not building

- PCB editor or auto-repair
- Every CAD and Gerber dialect
- Electrical, signal, thermal checks
- Stackup, BOM, stencil or assembly apps
- Checkout, payments or tracking
- Generic workflow platform

## Closing thought

What I would de-risk first: parser fidelity and the mapping from a finding to the exact geometry.

If a highlighted clearance lands on the wrong trace, nothing else in the product matters.

The rest of the system is conventional; that mapping is where trust is won or lost.

