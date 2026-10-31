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

- Trace order context: customer, quantity, board size, revision.
- Package files with detected layer and mapping state.
- Board preview while the package is being normalized.
- Detected products: PCB fabrication and solder-paste stencil.
- Processing timeline, the assigned manufacturing profile and the spoke ruleset.
- Interpretation notes for files that need attention.

### Screen 2: validation workspace

- Large 2D board viewer with layer visibility controls.
- Findings panel grouped by technical severity, with checkout impact shown separately.
- Selecting a finding highlights and zooms to its geometry.
- Finding details: explanation, measured value, required threshold, affected layer, rule id, and suggested action.
- Filters for technical severity and checkout impact.

### Screen 3: finding resolution

This is a focused state of the workspace, not a separate editor.

- Zoomed geometry with the measured gap marked.
- Rule explanation, net, layer, location, measured value, required value, and margin.
- Plain instruction for what to change in the EDA tool.
- Revision history, with original files still owned by Trace.
- Upload a revised package and revalidate. A fix only closes when a new run confirms it.

### Screen 4: results and handoff

- Validation summary: technical severity, checkout impact, order items, board count.
- Findings posted to Trace.
- Order items with kind, spec, quantity, cost, and posted or pending status.
- Clear return-to-order action, and upload of a corrected package.
- Run details: revision hash, assigned profile, spoke ruleset, parser versions.
