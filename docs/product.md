# Product

The PCB Validator is a Trace spoke. A customer opens it from an order, inspects the design package, reviews manufacturability findings, fixes the board in their EDA tool, uploads a revised package, and revalidates. When the run is complete, findings and priceable order items return to Trace.

The validator does not edit the PCB. It interprets the package, evaluates normalized geometry against the assigned manufacturing profile, and makes the result traceable.

## Package overview

The customer sees what the spoke received and what it understood before trusting any finding.

| Feature | Customer capability | How it works |
| --- | --- | --- |
| Package files | Inspect every uploaded file and its detected type. | The Spoke API reads the package manifest from Trace and maps each file to a role. |
| Copper layer detection | See all copper layers, including inner layers, not just top and bottom. | Identifies copper layers and maps them to stack positions. Incomplete or ambiguous mappings prevent affected checks from running. |
| Mask, silkscreen, outline and drill | Confirm the non-copper inputs used by the checks. | Each file is classified and normalized; the drill file is parsed separately from the Gerber layers. |
| Uncertainty and warnings | See what the package does not prove, such as unknown drill plating or a missing paste layer. | The spoke surfaces the ambiguity instead of silently assuming a manufacturing value. Affected items stay pending until the input or an approved fallback exists. |
| Processing state | Follow the run from file reading to layer mapping, geometry normalization and rule evaluation. | The worker reports progress to the Spoke API, which streams it to the browser over SSE. |
| Assigned manufacturing profile | See the profile and ruleset version used for the run. | Trace supplies the assigned profile. The spoke owns and versions the DFM rules that evaluate against it. Both are read-only to the customer. |

## Board inspection

The board is the reference for every finding. The customer can move between the list and the geometry without re-running validation.

| Feature | Customer capability | How it works |
| --- | --- | --- |
| Board rendering | See the board as a rendered design rather than a file list. | The server sends normalized geometry to the browser, which renders it with Canvas or WebGL. The browser never parses Gerber. |
| Layer control | Toggle top, inner 1, inner 2, bottom, soldermask, silkscreen, drill and outline. | Layer visibility is a local rendering state. The underlying geometry is already normalized and cached. |
| Zoom, pan and fit | Inspect dense areas and return to the full board. | Interaction happens in the browser and does not trigger a server round trip. |
| Finding overlays | See where each finding sits on the board. | Findings carry a geometry reference, so the renderer can place and highlight them directly. |
| Finding selection | Move from a list item to the exact violating geometry. | Selecting a finding focuses and zooms the board; the finding detail stays in sync. |
| Finding detail | Read the rule, measured value, required threshold, layer and affected geometry. | The finding record is stored with the run and linked to the rule and geometry that produced it. |
| Severity grouping | Separate findings by technical severity: errors, warnings and informational results. | The spoke reports technical severity. Trace decides which findings block checkout. |

## Finding resolution

A finding is useful only if the customer understands the constraint and knows what to change.

| Feature | Customer capability | How it works |
| --- | --- | --- |
| Plain explanation | Understand what the rule found without reading a parser log. | Each finding carries a short explanation, the rule id, the measured value, the required threshold and the margin. |
| Geometry focus | See the exact trace, pad, drill or mask feature involved. | The finding references normalized geometry and board coordinates, so the viewer can highlight and zoom to it. |
| Suggested remediation | Get guidance on the kind of change that would satisfy the constraint. | The note describes the constraint and points to the relevant routing or geometry. It is guidance, not an automated design instruction. |
| Acknowledgement | Acknowledge a finding, mark it to address, or request a manufacturing review. | These are tracking states. They do not close the finding and they do not change the order. |
| Resolution | Know when a finding is actually fixed. | A finding closes only when a later validation run confirms the geometry is no longer present. A checkbox cannot resolve it. |
| No in-app editing | Keep the design authority in the EDA tool. | The validator never edits geometry. It returns the customer to their design tool with the constraint clearly identified. |

## Areas to cover

- Revision workflow
- Results and handoff
