# Scope

## Core workflow

1. Customer launches the PCB Validator from a Trace order.
2. Spoke retrieves the order and its design package through the Hub API.
3. Spoke interprets the package and renders the board.
4. Spoke runs manufacturability checks against the assigned manufacturing profile.
5. Customer reviews findings and fixes the design in their EDA tool.
6. Customer uploads a revised package and revalidates.
7. Spoke publishes findings and priceable items to Trace.
8. Customer returns to the order and continues checkout.

## Package interpretation and validation

Interpretation identifies files, layers, geometry and specifications.

Validation evaluates normalized geometry against manufacturing rules.

These are separate logical stages, even if initially implemented in one processing pipeline.
