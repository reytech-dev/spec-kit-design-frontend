# Design Frontend Update Report

## Metadata

| Field | Value |
|---|---|
| Project | {{PROJECT}} |
| Frontend repository | {{FRONTEND_REPO_PATH}} |
| Design context | {{DESIGN_CONTEXT_PATH}} |
| Timestamp | {{TIMESTAMP}} |

## Screen Summary

| Category | Count |
|---|---|
| Unchanged | {{UNCHANGED_COUNT}} |
| Changed | {{CHANGED_COUNT}} |
| New | {{NEW_COUNT}} |
| Removed from design | {{REMOVED_COUNT}} |
| Unknown | {{UNKNOWN_COUNT}} |
| **Total** | {{TOTAL_COUNT}} |

## Updated Files

{{#UPDATED_FILES}}
- `{{path}}`
{{/UPDATED_FILES}}

## Route Map Updates

- Entries with updated `implementationRoute`: {{ROUTE_MAP_ENTRIES_UPDATED}}

{{#ROUTE_MAP_DETAILS}}
- `{{screenId}}` → `{{implementationRoute}}`
{{/ROUTE_MAP_DETAILS}}

## Verification

| Script | Status | Notes |
|---|---|---|
| codegen | {{CODEGEN_STATUS}} | {{CODEGEN_NOTES}} |
| lint | {{LINT_STATUS}} | {{LINT_NOTES}} |
| test | {{TEST_STATUS}} | {{TEST_NOTES}} |
| build | {{BUILD_STATUS}} | {{BUILD_NOTES}} |

## Next Steps

```bash
./bin/oe frontend:start {{FRONTEND_REPO}}
./bin/oe speckit:visual {{PROJECT}} compare --frontend-url http://node-runner:5173 --no-fail-on-diff
```

## Removed-from-Design Screens

The following screens exist in the frontend but have no corresponding entry in the current design contract. They have **not** been deleted. Manual review is recommended.

{{#REMOVED_SCREENS}}
- {{screenName}} (`{{screenId}}`) — route: `{{implementationRoute}}`
{{/REMOVED_SCREENS}}

{{^REMOVED_SCREENS}}
None.
{{/REMOVED_SCREENS}}
