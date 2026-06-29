---
description: Update an existing frontend repository from processed design artifacts.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding. If `--help` is passed,
show the usage below and stop.

## Usage

```text
/speckit.design-frontend.update --project <project-slug> --frontend-repo <frontend-repo> [flags]
```

### Required

| Flag | Description |
|---|---|
| `--project <slug>` | Project slug matching `workspace/design-context/<project>` |
| `--frontend-repo <repo>` | Frontend repository name matching `workspace/frontend/<frontend-repo>` |

### Optional

| Flag | Default | Description |
|---|---|---|
| `--dry-run` | `false` | Analyze inputs and produce an update plan only. Do not modify any files. |
| `--static-data` | `true` | Create or update static GraphQL fixtures for deterministic visual states. |
| `--no-static-data` | — | Do not create or update static GraphQL fixtures unless they already exist. |
| `--preserve-production-behavior` | `true` | Preserve existing real application behavior, auth, routing, GraphQL integration, and business workflows. |
| `--allow-package-changes` | `false` | Permit `package.json` changes if a missing dependency or script is required. |
| `--update-route-map` | `true` | Fill missing `implementationRoute` values for newly created screens in `route-map.json`. |
| `--no-update-route-map` | — | Do not modify `route-map.json`. |
| `--visual-target` | `true` | Optimize changes toward passing visual comparison. |
| `--max-files <number>` | none | Safety cap for number of files the command may modify. |
| `--help` | — | Show usage and stop. |

## Domain Boundary

This command owns:

- Applying processed design artifacts to existing frontend repositories.
- Updating routes, components, styles, design references, and static GraphQL fixtures.
- Generating a design-to-frontend update plan.
- Preserving production application behavior while aligning changed/new screens visually.

This command does **not** own:

- Crawling Open Design exports.
- Running Playwright capture against the design prototype.
- Generating `design-ir.json`.
- Generating `source-map.json`.
- Generating screenshots.
- Processing raw Open Design exports.
- Creating `frontend-staging` from the UI blueprint.
- Cloning `ui-apollo-blueprint`.
- Replacing or resetting production frontend repositories.

Those remain separate responsibilities belonging to `speckit.open-design.process` and the `opencode-environment` visual comparison runner.

## Prerequisites

This command must run **after** `/speckit.open-design.process`. Design artifacts must already exist before this command can operate.

Visual comparison is executed separately by `opencode-environment`, not by this command.

---

## Phase 1: Parse Arguments and Validate Paths

### 1.1 Extract flags from `$ARGUMENTS`

Parse `--project`, `--frontend-repo`, `--dry-run`, `--static-data`, `--no-static-data`, `--preserve-production-behavior`, `--allow-package-changes`, `--update-route-map`, `--no-update-route-map`, `--visual-target`, `--max-files`, `--help`.

Mutually exclusive:
- `--static-data` and `--no-static-data`
- `--update-route-map` and `--no-update-route-map`

If `--help` is present, print the usage table above and stop immediately.

### 1.2 Validate `--project` and `--frontend-repo`

Both are required. If either is missing, respond:

```text
Both --project and --frontend-repo are required.

Usage:
  /speckit.design-frontend.update --project <project-slug> --frontend-repo <frontend-repo> [flags]
```

### 1.3 Resolve paths

```text
WORKSPACE     =  workspace/
DESIGN_CTX    =  workspace/design-context/<project>
FRONTEND_REPO =  workspace/frontend/<frontend-repo>
```

### 1.4 Validate required design artifacts

Check that every required file exists:

```text
workspace/design-context/<project>/design-processing/design-ir.json
workspace/design-context/<project>/design-processing/design-tokens.json
workspace/design-context/<project>/design-processing/component-contracts.md
workspace/design-context/<project>/design-processing/page-structures.md
workspace/design-context/<project>/design-processing/data-mappings.json
workspace/design-context/<project>/design-processing/frontend-implementation-brief.md
workspace/design-context/<project>/visual-regression/fixtures/source-map.json
workspace/design-context/<project>/visual-regression/fixtures/route-map.json
workspace/design-context/<project>/visual-regression/screenshots/
workspace/frontend/<frontend-repo>/package.json
```

If any required file is missing, respond:

```text
Design frontend update cannot run because processed design artifacts are missing.

Run:

  ./bin/oe speckit:visual <project> all

Then inside the agent:

  /speckit.open-design.process --project <project>

Then rerun:

  /speckit.design-frontend.update --project <project> --frontend-repo <frontend-repo>
```

### 1.5 Validate frontend repository

Check that `workspace/frontend/<frontend-repo>/package.json` exists and is readable.

If the frontend repository directory does **not** exist at all, respond:

```text
Frontend repository not found:

  workspace/frontend/<frontend-repo>

This command updates an existing frontend repository.
For first-time implementation, create frontend staging first and later copy it to workspace/frontend/<frontend-repo>.
```

### 1.6 Reject forbidden paths

The command must **never** read from or write to:

```text
workspace/frontend-staging/**
```

If a path under `workspace/frontend-staging/` is encountered during execution, ignore it.

The command must **never** write to:

```text
workspace/design-context/<project>/index.html
workspace/design-context/<project>/visual-regression/screenshots/**
workspace/design-context/<project>/design-processing/design-ir.json
workspace/design-context/<project>/design-processing/design-tokens.json
workspace/design-context/<project>/visual-regression/fixtures/source-map.json
```

The command may read these files but must never modify them.

The command must **never** clone or materialize `ui-apollo-blueprint`.

---

## Phase 2: Load the Processed Design Contract

### 2.1 Read design artifacts

Read all of these files and build an internal model:

```text
design-ir.json          — screen definitions, component tree, layout/styling information
design-tokens.json      — design tokens (colors, spacing, typography, etc.)
component-contracts.md  — expected component interfaces and responsibilities
page-structures.md      — page-level layout and content structure
data-mappings.json      — expected data shapes, queries, field mappings
frontend-implementation-brief.md — implementation-level guidance for each screen
source-map.json         — maps design screens to source file expectations and viewport variants
route-map.json          — maps screen IDs to expected implementation routes
```

Read the screenshots directory to understand which screens have reference images:

```text
workspace/design-context/<project>/visual-regression/screenshots/
```

### 2.2 Read optional files

If present, read:

```text
workspace/design-context/<project>/design-processing/change-summary.md
```

This file describes what changed between design iterations and is valuable for classification.

### 2.3 Build internal design model

Construct an internal representation containing for each screen:

- `screenId` — stable identifier from `design-ir.json`
- `screenName` — human-readable name
- `viewportVariants` — viewport sizes the screen targets (from `source-map.json`)
- `referenceScreenshots` — paths to screenshot files under `screenshots/`
- `implementationRoute` — expected URL path (from `route-map.json`, may be empty for new screens)
- `expectedComponents` — components the screen requires (from `component-contracts.md`)
- `layoutRequirements` — layout constraints and patterns (from `page-structures.md`)
- `tokenRequirements` — design tokens used (from `design-tokens.json`)
- `dataRequirements` — static fixture data shapes (from `data-mappings.json`)
- `implementationNotes` — guidance from `frontend-implementation-brief.md`
- `changeNotes` — change information from `change-summary.md` if present

---

## Phase 3: Inspect the Existing Frontend Repository

### 3.1 Read frontend metadata

Read:

```text
workspace/frontend/<frontend-repo>/AGENTS.md           (if present)
workspace/frontend/<frontend-repo>/README.speckit.md    (if present)
workspace/frontend/<frontend-repo>/package.json
```

### 3.2 Explore frontend structure

Read directory listings for:

```text
workspace/frontend/<frontend-repo>/src/
workspace/frontend/<frontend-repo>/src/routes/
workspace/frontend/<frontend-repo>/src/components/
workspace/frontend/<frontend-repo>/src/graphql/
workspace/frontend/<frontend-repo>/src/mocks/
workspace/frontend/<frontend-repo>/src/design/
```

### 3.3 Detect frontend conventions

Identify:

- **Routing framework or route conventions**: How are routes registered? File-system based? Config-based? What framework?
- **Component organization**: How are components structured? What file extensions? What export conventions?
- **Style organization**: CSS modules? Tailwind? Styled-components? What file conventions?
- **GraphQL client setup**: What client library? Where are operations defined? How are queries structured?
- **Static fixture setup**: Where are mock handlers? How are fixtures organized? What library (MSW, etc.)?
- **Existing design-derived files**: Does `src/design/` already exist? What files are present?
- **Existing screen/component ownership**: Which screens already have route files? Which components exist?

### 3.4 Read existing design references

If `workspace/frontend/<frontend-repo>/src/design/` exists, read any existing:

```text
src/design/design-ir.json
src/design/design-tokens.json
src/design/source-map.json
src/design/route-map.json
src/design/frontend-implementation-brief.md
```

These are previous snapshots that can help with change detection.

Do **not** assume the frontend still looks like the original UI blueprint. The frontend may have been modified since its last design update.

---

## Phase 4: Classify Design Changes

### 4.1 Match screens between design and frontend

For each screen in the design model, attempt to match it to an existing frontend screen using this priority order:

1. **route-map entry id**: Match `route-map.json` entries by their `id` field to the design `screenId`.
2. **screenId**: Direct match of `design-ir.json` screen identifiers to existing route/component names.
3. **implementationRoute**: Match by existing route paths in the frontend route configuration.
4. **Stable screen name**: Match by human-readable screen name similarity.
5. **Existing `src/design/source-map.json`**: If present, use its mappings as a fallback.
6. **Agent judgment**: When no clear match exists, use reasonable heuristics (similar page structure, shared components, typical naming patterns).

### 4.2 Classify each screen

Assign every design screen to exactly one of these categories:

| Category | Definition |
|---|---|
| `unchanged` | Screen exists in frontend and its design contract has not meaningfully changed since last update. |
| `changed` | Screen exists in frontend but design contract shows updates to layout, components, data, styling, or route. |
| `new` | Screen exists in the design contract but has no matching screen in the frontend. |
| `unknown` | Cannot confidently classify. Treat with caution and report as risky. |

### 4.3 Identify removed-from-design screens

Compare the frontend's actual screens against the design contract. Any screen that exists in the frontend but has no corresponding entry in the design contract is `removed-from-design`.

**Critical rule for removed-from-design screens:**

- Do **not** delete production code.
- Do **not** remove routes automatically.
- Do **not** remove tests automatically.
- Report them as `removed-from-design` in the plan and final report.
- The decision to delete belongs to the development team, not this command.

---

## Phase 5: Generate an Update Plan

### 5.1 Produce the plan

Before modifying any file, construct an update plan containing:

```text
- project slug
- frontend repository path
- design context path
- classification summary (unchanged, changed, new, removed-from-design, unknown counts)
- per-screen breakdown with matched status and planned actions
- files expected to be modified (with reason for each)
- route-map entries expected to be updated (screen ID → new implementationRoute)
- risks and assumptions
```

### 5.2 Write the plan file

Always write the plan to:

```text
workspace/design-context/<project>/handoff/design-frontend-update-plan.md
```

Create the `handoff/` directory if it does not exist.

### 5.3 Print a concise summary

Output a summary to the user:

```text
Design frontend update plan

Project: <project>
Frontend: workspace/frontend/<frontend-repo>
Design context: workspace/design-context/<project>

Screens:
  unchanged: <n>
  changed: <n>
  new: <n>
  removed-from-design: <n>
  unknown: <n>

Files expected to be modified: <n>
Route-map entries expected to be updated: <n>

Plan written to:
  workspace/design-context/<project>/handoff/design-frontend-update-plan.md
```

### 5.4 Dry-run stop

If `--dry-run` is set, **stop here**. Do not modify any frontend files. Do not update `src/design/`. Do not update `route-map.json`. The plan file and summary are the only outputs.

---

## Phase 6: Apply Changes to the Frontend Repository

### 6.1 Safety cap

If `--max-files <number>` is set and the planned modifications exceed the cap, respond:

```text
Update aborted: expected <n> file modifications but --max-files is set to <cap>.

To proceed, increase --max-files or review the plan:
  workspace/design-context/<project>/handoff/design-frontend-update-plan.md
```

### 6.2 Update changed screens

For each `changed` screen:

1. **Read the existing route/component/style files** completely before editing.
2. **Make targeted edits** to align with the design contract:
   - Update component structure to match `component-contracts.md`.
   - Update layout to match `page-structures.md`.
   - Update styles using tokens from `design-tokens.json`.
   - Update data bindings from `data-mappings.json`.
3. **Update static fixtures** when visual data changed and `--static-data` is not overridden by `--no-static-data`. Only add or update fixture files under `src/mocks/graphql/` or the frontend's existing fixture location.
4. **Update `src/design/` references** by copying the latest design artifacts (see Phase 8).
5. **Preserve existing production behavior**:
   - Do not remove auth guards.
   - Do not remove real GraphQL client integration.
   - Do not replace production data flows with static mocks globally.
   - Keep routing conventions intact.
   - Do not rewrite the application shell.

Prefer small, targeted edits over broad rewrites.

### 6.3 Add new screens

For each `new` screen:

1. **Create route/component files** following the existing frontend conventions detected in Phase 3:
   - Match the routing framework (file-system router, config-based, etc.).
   - Match component patterns (file extensions, export style, props conventions).
   - Match style organization (CSS modules, Tailwind classes, styled-components, etc.).
   - Follow the same directory nesting conventions.
2. **Create a stable implementation route** based on the screen name and existing route patterns. Use kebab-case URL paths consistent with the frontend's existing routes.
3. **Add static GraphQL fixture files** if `--static-data` is enabled (and not overridden by `--no-static-data`). Place fixtures in the frontend's existing mocks directory following its conventions. Use data shapes from `data-mappings.json`.
4. **Register the new route** in the frontend's route configuration following its existing registration pattern.
5. **Update `route-map.json`** if `--update-route-map` is enabled (and not overridden by `--no-update-route-map`). For each new screen, add or update its entry:

```json
{
  "screenId": "<screen-id>",
  "implementationRoute": "/the-new-route"
}
```

Write to:

```text
workspace/design-context/<project>/visual-regression/fixtures/route-map.json
```

Do **not** modify `source-map.json` or any other fixture file.

6. **Add minimal tests** if the frontend repository has an obvious test convention (e.g., co-located `*.test.ts` files, a `__tests__/` directory, etc.). Follow the existing test patterns exactly.

### 6.4 Handle removed-from-design screens

For each `removed-from-design` screen:

- Do **not** delete any files.
- Do **not** remove routes.
- Do **not** remove tests.
- Include them in the plan and final report as `removed-from-design` with a note that deletion is a manual decision.

### 6.5 Preserve production behavior

When `--preserve-production-behavior` is enabled (default):

- Do **not** remove or disable auth guards.
- Do **not** remove or bypass real GraphQL client integration.
- Do **not** replace production data flows globally with static mocks.
- Do **not** delete existing routes.
- Do **not** delete existing business logic.
- Do **not** weaken visual comparison thresholds.
- Do **not** rewrite the application shell unnecessarily.

Static fixtures may be added or updated **only** for deterministic visual states when `--static-data` is enabled. They must coexist with real data flows, not replace them.

### 6.6 Package changes

Only modify `package.json` if `--allow-package-changes` is explicitly set. If a missing dependency or script is required and this flag is not set, report it as a risk in the plan and skip the change.

---

## Phase 7: Update Local Design References

### 7.1 Ensure `src/design/` directory exists

```text
workspace/frontend/<frontend-repo>/src/design/
```

Create the directory if it does not exist.

### 7.2 Copy design reference files

Copy the latest versions of these files from the design context into the frontend repo:

```text
From: workspace/design-context/<project>/design-processing/design-ir.json
To:   workspace/frontend/<frontend-repo>/src/design/design-ir.json

From: workspace/design-context/<project>/design-processing/design-tokens.json
To:   workspace/frontend/<frontend-repo>/src/design/design-tokens.json

From: workspace/design-context/<project>/visual-regression/fixtures/source-map.json
To:   workspace/frontend/<frontend-repo>/src/design/source-map.json

From: workspace/design-context/<project>/visual-regression/fixtures/route-map.json
To:   workspace/frontend/<frontend-repo>/src/design/route-map.json

From: workspace/design-context/<project>/design-processing/frontend-implementation-brief.md
To:   workspace/frontend/<frontend-repo>/src/design/frontend-implementation-brief.md
```

These are local copies for development convenience. The canonical source remains:

```text
workspace/design-context/<project>
```

---

## Phase 8: Verification

### 8.1 Detect available scripts

Read `package.json` from the frontend repository and check for these scripts:

```text
codegen
lint
test
build
```

### 8.2 Install dependencies if needed

If `node_modules/` is missing inside the frontend repository, run:

```bash
pnpm install
```

Use `npm` or `yarn` if the frontend repository has a lock file indicating a different package manager (`package-lock.json` → npm, `yarn.lock` → yarn). If no lock file is present, default to `pnpm`.

### 8.3 Run verification scripts

Run each detected script in this order:

1. `pnpm codegen` (GraphQL code generation, if available)
2. `pnpm lint` (linting, if available)
3. `pnpm test` (tests, if available)
4. `pnpm build` (production build, if available)

For any script that does not exist, skip it and note "skipped" in the report.

Report each as: `passed`, `failed`, or `skipped`.

If a script fails, report the failure but continue with remaining scripts.

### 8.4 Handle failures

If `pnpm build` fails, log the error output (first 50 lines) and include it in the final report. Do not attempt to fix build errors automatically — report them for the developer to address.

---

## Phase 9: Final Output

After all modifications and verification are complete, print:

```text
Design frontend update complete.

Project:
  <project>

Frontend repository:
  workspace/frontend/<frontend-repo>

Design context:
  workspace/design-context/<project>

Screens:
  unchanged: <n>
  changed: <n>
  new: <n>
  removed-from-design: <n>
  unknown: <n>

Updated files:
  <list each modified file path>

Route map:
  updated implementationRoute values: <n>

Verification:
  pnpm codegen: passed/skipped/failed
  pnpm lint: passed/skipped/failed
  pnpm test: passed/skipped/failed
  pnpm build: passed/skipped/failed

Next:
  ./bin/oe frontend:start <frontend-repo>
  ./bin/oe speckit:visual <project> compare --frontend-url http://node-runner:5173 --no-fail-on-diff
```

Also write this report to:

```text
workspace/design-context/<project>/handoff/design-frontend-update-report.md
```

---

## Path Rules Summary

### May read

```text
workspace/design-context/<project>/**
workspace/frontend/<frontend-repo>/**
```

### May modify

```text
workspace/frontend/<frontend-repo>/src/**
workspace/frontend/<frontend-repo>/tests/**
workspace/frontend/<frontend-repo>/e2e/**
workspace/frontend/<frontend-repo>/package.json                (only with --allow-package-changes)
workspace/frontend/<frontend-repo>/README.speckit.md
workspace/design-context/<project>/visual-regression/fixtures/route-map.json  (only with --update-route-map)
workspace/frontend/<frontend-repo>/src/design/**               (local design reference copies)
workspace/design-context/<project>/handoff/**                  (plan and report files)
```

### Must never modify

```text
workspace/design-context/<project>/index.html
workspace/design-context/<project>/visual-regression/screenshots/**
workspace/design-context/<project>/design-processing/design-ir.json
workspace/design-context/<project>/design-processing/design-tokens.json
workspace/design-context/<project>/visual-regression/fixtures/source-map.json
workspace/frontend-staging/**
```

---

## Error Messages

### Missing design artifacts

```text
Design frontend update cannot run because processed design artifacts are missing.

Run:

  ./bin/oe speckit:visual <project> all

Then inside the agent:

  /speckit.open-design.process --project <project>

Then rerun:

  /speckit.design-frontend.update --project <project> --frontend-repo <frontend-repo>
```

### Frontend repository not found

```text
Frontend repository not found:

  workspace/frontend/<frontend-repo>

This command updates an existing frontend repository.
For first-time implementation, create frontend staging first and later copy it to workspace/frontend/<frontend-repo>.
```

### Missing --project or --frontend-repo

```text
Both --project and --frontend-repo are required.

Usage:
  /speckit.design-frontend.update --project <project-slug> --frontend-repo <frontend-repo> [flags]
```

### --max-files exceeded

```text
Update aborted: expected <n> file modifications but --max-files is set to <cap>.

To proceed, increase --max-files or review the plan:
  workspace/design-context/<project>/handoff/design-frontend-update-plan.md
```

---

## Quality Checklist

Before returning the final report, verify internally that:

- [ ] Required design artifacts were all found and read.
- [ ] Frontend repository was found and inspected.
- [ ] Screen classification was applied to every design screen.
- [ ] `removed-from-design` screens were reported but not deleted.
- [ ] Update plan was written to `handoff/design-frontend-update-plan.md`.
- [ ] If `--dry-run`, no frontend files were modified.
- [ ] Changed screens received targeted edits, not broad rewrites.
- [ ] New screens followed existing frontend conventions.
- [ ] `route-map.json` was only updated when `--update-route-map` is enabled (and not overridden).
- [ ] `package.json` was only modified when `--allow-package-changes` is set.
- [ ] Design reference screenshots were never modified.
- [ ] `frontend-staging/` was never read from or written to.
- [ ] `ui-apollo-blueprint` was never cloned or materialized.
- [ ] Verification scripts were detected and run.
- [ ] Final report includes all counts, file lists, and the next visual comparison command.
- [ ] Report was written to `handoff/design-frontend-update-report.md`.
- [ ] Production behavior was preserved (auth, GraphQL, routing, business logic intact).
