# Design Frontend Update Plan

## Metadata

| Field | Value |
|---|---|
| Project | {{PROJECT}} |
| Frontend repository | {{FRONTEND_REPO_PATH}} |
| Design context | {{DESIGN_CONTEXT_PATH}} |
| Timestamp | {{TIMESTAMP}} |
| Dry run | {{DRY_RUN}} |

## Screen Classification

| Category | Count |
|---|---|
| Unchanged | {{UNCHANGED_COUNT}} |
| Changed | {{CHANGED_COUNT}} |
| New | {{NEW_COUNT}} |
| Removed from design | {{REMOVED_COUNT}} |
| Unknown | {{UNKNOWN_COUNT}} |
| **Total** | {{TOTAL_COUNT}} |

## Per-Screen Details

{{#SCREENS}}

### {{screenName}} (`{{screenId}}`)

- **Classification**: {{classification}}
- **Implementation route**: `{{implementationRoute}}`
- **Viewport variants**: {{viewportVariants}}
- **Planned actions**: {{plannedActions}}
- **Reference screenshots**: {{referenceScreenshots}}

{{/SCREENS}}

## Files Expected to be Modified

| File | Reason |
|---|---|
{{#FILES}}
| {{path}} | {{reason}} |
{{/FILES}}

## Route Map Updates Expected

| Screen ID | New implementationRoute |
|---|---|
{{#ROUTE_MAP_UPDATES}}
| {{screenId}} | {{implementationRoute}} |
{{/ROUTE_MAP_UPDATES}}

## Risks and Assumptions

{{RISKS_AND_ASSUMPTIONS}}

## Notes

- Removed-from-design screens are reported but will not be deleted automatically.
- Production behavior (auth, routing, GraphQL integration, business logic) will be preserved.
- Static fixtures will only be added/updated for deterministic visual states.
- Design reference screenshots will not be modified.
- `frontend-staging/` will not be created or modified.
- `ui-apollo-blueprint` will not be cloned.
