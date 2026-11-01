# Mockups

The customer fixes the PCB in their own EDA tool. The spoke never edits geometry, so there is no board editor in any screen.

## Screens

| Screen | Purpose |
| --- | --- |
| 1\. Package overview | Order context, files, detected products and processing status |
| 2\. Validation workspace | Board viewer, layers, findings and finding details |
| 3\. Finding resolution | Selected issue, geometry highlight, explanation and corrective action |
| 4\. Results and handoff | Validation summary, generated order items, findings posted to Trace and return to order |

### Screen 1: package overview

Order context, files, detection and processing status.

This screen is the first trust boundary. The order supplies a four-layer package, so all four copper layers are listed and mapped explicitly. Anything the package does not prove, such as drill plating or a missing paste layer, is shown as a question instead of silently assumed. The manufacturing profile is assigned to the order and the ruleset belongs to the spoke, so the customer sees both versions but cannot change them here. Uploading a revised package adds a revision to the order; it does not replace the current revision or its results. The board on this screen is illustrative until normalization finishes; the viewer renders parsed geometry after the run.

- Trace order context: customer, quantity, board size, revision.
- Package files with detected layer and mapping state.
- Board preview while the package is being normalized.
- Detected products: PCB fabrication and solder-paste stencil.
- Processing timeline, the assigned manufacturing profile and the spoke ruleset.
- Interpretation notes for files that need attention.

### Screen 2: validation workspace

Board viewer, layer control and the findings wall.

The board and the findings stay in one view, because the customer needs to compare a measurement with the geometry that produced it. Selecting a finding is a local action: it highlights and zooms the board without re-running validation. Findings are grouped by technical severity, while checkout impact is shown separately because Trace owns that policy. The profile belongs to the order and the ruleset belongs to the spoke, so both are read-only here.

- Large 2D board viewer with layer visibility controls.
- Findings panel grouped by technical severity, with checkout impact shown separately.
- Selecting a finding highlights and zooms to its geometry.
- Finding details: explanation, measured value, required threshold, affected layer, rule id, and suggested action.
- Filters for technical severity and checkout impact.

### Screen 3: finding resolution

One issue, its geometry and what to change in the EDA tool.

This is a focused state of the workspace, not a separate editor, and the customer still fixes the board in their EDA tool. The finding states the constraint, the measured geometry, and a suggested remediation framed as guidance. Acknowledgement is tracked separately from resolution: a finding only closes when a later validation run confirms it, never because someone checked a box. Trace owns the original files and revision manifest. The spoke stores the revision reference, content hash, and derived results for each run.

- Zoomed geometry with the measured gap marked.
- Rule explanation, net, layer, location, measured value, required value, and margin.
- Plain instruction for what to change in the EDA tool.
- Revision history, with original files still owned by Trace.
- Upload a revised package and revalidate. A fix only closes when a new run confirms it.

### Screen 4: results and handoff

Findings posted back to Trace, order items posted, and return to the order.

The spoke reports and the hub decides. A run can complete while findings that block checkout remain, so checkout eligibility stays a hub policy rather than something the validator grants. The spoke posts findings and the priceable items it can actually derive from the package, keeps the derived data needed to re-render the board, and leaves cost roll-up, checkout and production tracking to Trace.

- Validation summary: technical severity, checkout impact, order items, board count.
- Findings posted to Trace.
- Order items with kind, spec, quantity, cost, and posted or pending status.
- Clear return-to-order action, and upload of a corrected package.
- Run details: revision hash, assigned profile, spoke ruleset, parser versions.
