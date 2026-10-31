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
