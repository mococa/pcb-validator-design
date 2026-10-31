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

## Areas to cover

- Board inspection
- Finding resolution
- Revision workflow
- Results and handoff
