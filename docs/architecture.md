# PCB Validator Architecture

## Overview

The PCB Validator uses a hub-and-spoke architecture.

- **Trace** owns identity, orders, original package files, revisions, assigned manufacturing profiles, cost roll-up, and checkout.
- **The PCB Validator spoke** owns package interpretation, normalized geometry, validation runs, rules, findings, and derived artifacts.
- **The browser** displays normalized geometry and validation results. It does not parse Gerber files.

## System Components

### Browser

**Technology:** Preact / TypeScript

- Render PCB boards and layers.
- Navigate validation findings.
- Upload revised packages.
- Display validation progress.
- Display completed validation results and normalized geometry.

### Spoke API

**Technology:** FastAPI / Cloud Run

- Retrieve order context and package manifests.
- Retrieve the assigned manufacturing profile from Trace.
- Manage validation runs and their lifecycle.
- Manage findings and revisions.
- Dispatch validation jobs.
- Serve normalized geometry.
- Stream validation progress and completion through Server-Sent Events (SSE).
- Persist run state and findings.

### Validation Worker

**Technology:** Python / Cloud Run Job

- Fetch package files from Trace.
- Parse Gerber and Excellon files.
- Normalize PCB geometry.
- Execute the Design for Manufacturability (DFM) rules.
- Write derived geometry to object storage.
- Report result metadata and findings to the Spoke API.

> The worker never accesses the database directly.

### Spoke Database

**Technology:** PostgreSQL

Stores validation runs, revision records, findings and processing state.

### Object Storage

Stores normalized PCB geometry and larger validation artifacts.

### Trace Hub API

Provides order context, package manifests and assigned manufacturing profiles. Receives findings and priceable order items.

## Main Data Flows

1. **Request order context:** The browser requests order context from the Spoke API.
2. **Retrieve order information:** The Spoke API retrieves order context, the package manifest, and the assigned manufacturing profile from Trace.
3. **Dispatch validation job:** The Spoke API dispatches a validation job containing the order ID, revision ID, and file references.
4. **Fetch package files:** The worker fetches the actual package files from Trace.
5. **Process package:** The worker parses Gerber and Excellon files, normalizes geometry, and executes the DFM rules.
6. **Store geometry:** The worker writes derived geometry to object storage.
7. **Report validation results:** The worker reports result metadata and findings to the Spoke API.
8. **Persist validation state:** The API stores run state and findings in PostgreSQL.
9. **Retrieve geometry:** The API reads derived geometry from object storage when serving the browser.
10. **Stream progress:** The API streams validation progress and completion events to the browser through SSE.
11. **Return and display results:** The API returns validation results and geometry references to the browser.
