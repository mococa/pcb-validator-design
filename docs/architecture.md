# PCB Validator Architecture

## Overview

The PCB Validator uses a hub-and-spoke architecture.

- **Trace** owns identity, orders, original package files, revisions, assigned manufacturing profiles, cost roll-up, and checkout.
- **The PCB Validator spoke** owns package interpretation, normalized geometry, validation runs, rules, validation findings and their provenance, and derived artifacts.
- **The browser** displays normalized geometry and validation results. It does not parse Gerber files.

## System Components

### 1\. Browser

**Technology:** Preact / TypeScript

**Responsibilities:**

- Render PCB boards and layers.
- Navigate validation findings.
- Upload revised packages.
- Display validation progress.
- Display completed validation results and normalized geometry.

### 2\. Spoke API

**Technology:** FastAPI / Cloud Run

**Responsibilities:**

- Retrieve order context and package manifests.
- Retrieve the assigned manufacturing profile from Trace; apply the spoke-owned DFM ruleset.
- Manage validation runs and their lifecycle.
- Manage findings and revisions.
- Dispatch validation jobs.
- Serve normalized geometry.
- Stream validation progress and completion through Server-Sent Events (SSE).
- Persist run state and findings.
- Submit findings and priceable order items to Trace.

### 3\. Validation Worker

**Technology:** Python / Cloud Run Job

**Responsibilities:**

- Fetch package files from Trace using its own authorized Hub API access.
- Parse Gerber and Excellon files.
- Normalize PCB geometry.
- Execute the spoke-owned Design for Manufacturability (DFM) rules.
- Write derived geometry to object storage.
- Report result metadata and findings to the Spoke API.

**Processing sequence:**

1. Fetch package files.
2. Parse files.
3. Normalize geometry.
4. Execute validation rules.
5. Store derived geometry.
6. Report results to the API.

> The worker never accesses the database directly.

### 4\. Spoke Database

**Technology:** PostgreSQL

**Stores:**

- Validation runs.
- Revision records.
- Findings.
- Processing state.

### 5\. Object Storage

**Stores:**

- Normalized PCB geometry.
- Larger validation artifacts.

The worker writes derived artifacts to object storage. The API reads these artifacts when serving the browser.

### 6\. Trace Hub API

**Responsibilities:**

- Provide order context.
- Own original package files and revisions.
- Provide package manifests.
- Provide assigned manufacturing profiles.
- Receive findings and priceable order items.
- Manage order-item endpoints.

> Original package files remain in Trace.

## System Boundaries and Ownership

### Trace Hub owns

- Identity.
- Orders.
- Original package files.
- Package revisions.
- Assigned manufacturing profiles.
- Cost roll-up.
- Order-scoped findings and their checkout impact.
- Checkout.

### PCB Validator spoke owns

- Package interpretation.
- Geometry parsing and normalization.
- Validation rules.
- Validation runs.
- Validation findings, including computation, provenance and resolution across runs.
- Derived geometry.
- Validation artifacts.

### Browser owns

- Board rendering.
- Layer controls.
- Finding navigation.
- Revision upload.
- Result presentation.

### Validation worker owns

- File processing.
- Geometry normalization.
- DFM rule execution.
- Derived artifact generation.

The worker accesses Trace through its authorized Hub API access. It reports results to the Spoke API, which is responsible for database persistence.

## Main Data Flows

1. **Request order context:** The browser requests order context from the Spoke API.
2. **Retrieve order information:** The Spoke API retrieves order context, the package manifest, and the assigned manufacturing profile from Trace.
3. **Dispatch validation job:** The Spoke API dispatches a validation job containing the order ID, revision ID, and file references.
4. **Fetch package files:** The worker fetches the actual package files from Trace using its own authorized Hub API access.
5. **Process package:** The worker parses Gerber and Excellon files, normalizes geometry, and executes the spoke-owned DFM rules.
6. **Store geometry:** The worker writes derived geometry to object storage.
7. **Report validation results:** The worker reports result metadata and findings to the Spoke API.
8. **Persist validation state:** The API stores run state and findings in PostgreSQL.
9. **Retrieve geometry:** The API reads derived geometry from object storage when serving the browser.
10. **Stream progress:** The API streams validation progress and completion events to the browser through SSE.
11. **Return and display results:** The API returns validation results and geometry references or payloads to the browser, which displays the completed results.
12. **Upload revision:** The browser uploads a revised package to Trace. The customer triggers validation again from the spoke UI; the spoke reads the new revision through the Hub API and starts a new validation run.
13. **Submit findings:** After the run is persisted, the spoke posts findings and priceable order items to Trace. If Trace is unavailable, this post can be retried without rerunning validation.

## Important Architectural Rules

- Original package files and revisions remain in Trace.
- Derived geometry and validation artifacts belong to the spoke.
- The browser never parses Gerber or Excellon files.
- The browser renders normalized geometry provided by the spoke.
- The validation worker never accesses PostgreSQL directly.
- The validation worker communicates results through the Spoke API.
- The Spoke API owns persistence of validation state and findings.
- The worker uses its own authorized access to retrieve original files from Trace.
- Trace remains the source of truth for orders, identity, assigned manufacturing profiles, and checkout.
