# Scope

## Core workflow

1. Customer launches the PCB Validator from a Trace order.
2. Spoke retrieves the order and its design package through the Hub API.
3. Spoke interprets the package and renders the board.
4. Spoke runs manufacturability checks against the assigned manufacturing profile.
5. Customer reviews findings, identifies required changes and fixes the design in their EDA tool.
6. Customer uploads a revised package to the order and revalidates.
7. Spoke publishes findings and priceable order items to Trace.
8. Customer returns to the order and continues checkout.

## Package interpretation and validation

Interpretation identifies files, layers, geometry and specifications.

Validation evaluates normalized geometry against manufacturing rules.

These are separate logical stages, even if initially implemented in one processing pipeline.

## Ownership

### Trace Hub
- Identity, authentication and organization scope.
- Orders and order lifecycle.
- Original package files.
- Assigned manufacturing profiles.
- App registry and permissions.
- Order items, order-scoped findings, costing and checkout.
- Production tracking and evidence.

### PCB Validator Spoke
- PCB package interpretation.
- Layer identification and geometry normalization.
- Validation runs and processing history.
- DFM rules that evaluate against the assigned manufacturing profile.
- Validation findings, including computation, provenance and resolution across runs.
- Derived geometry and rendering artifacts.
- Validator UI.

### Technical split

Browser: rendering, layer interaction, finding navigation and user interaction.

Server: authoritative parsing, geometry normalization and DFM validation.
