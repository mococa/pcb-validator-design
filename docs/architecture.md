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
