# AGENTS.md

## Extension Identity

- **Extension ID**: `design-frontend`
- **Extension name**: Specify Design Frontend
- **Repository**: `https://github.com/reytech-dev/spec-kit-design-frontend`
- **Command**: `/speckit.design-frontend.update`

## Domain Boundary

### This extension owns

- Applying processed design artifacts to existing frontend repositories.
- Updating routes, components, styles, design references, and static GraphQL fixtures.
- Generating a design-to-frontend update plan (`design-frontend-update-plan.md`).
- Generating a final update report (`design-frontend-update-report.md`).
- Preserving production application behavior while aligning changed/new screens visually.
- Updating local frontend `src/design/` reference files.
- Optionally updating `route-map.json` implementationRoute values for new screens.

### This extension does NOT own

- Crawling Open Design exports.
- Running Playwright capture against the design prototype.
- Generating `design-ir.json`.
- Generating `source-map.json`.
- Generating screenshots.
- Processing raw Open Design exports.
- Creating `frontend-staging` from the UI blueprint.
- Cloning `ui-apollo-blueprint`.
- Replacing or resetting production frontend repositories.
- Running visual comparison (that belongs to `opencode-environment`).
- Deleting production screens automatically.

## Command

```text
/speckit.design-frontend.update --project <project-slug> --frontend-repo <frontend-repo> [flags]
```

See `commands/speckit.design-frontend.update.md` for the full agent instruction.

## Path Rules

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

### Must never read or write

```text
workspace/frontend-staging/**
```

### Must never clone

```text
ui-apollo-blueprint
```

## Required Inputs

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

## Recommended Optional Inputs

```text
workspace/design-context/<project>/design-processing/change-summary.md
workspace/frontend/<frontend-repo>/AGENTS.md
workspace/frontend/<frontend-repo>/README.speckit.md
workspace/frontend/<frontend-repo>/src/design/
workspace/frontend/<frontend-repo>/src/mocks/graphql/
```

## Flag Defaults

| Flag | Default |
|---|---|
| `--dry-run` | `false` |
| `--static-data` | `true` |
| `--preserve-production-behavior` | `true` |
| `--allow-package-changes` | `false` |
| `--update-route-map` | `true` |
| `--visual-target` | `true` |
| `--max-files` | none (unlimited) |

## Screen Classification

Screens are classified into one of:

- `unchanged` — exists in frontend and design contract has not meaningfully changed
- `changed` — exists in frontend but design contract shows updates
- `new` — in design contract but not in frontend
- `removed-from-design` — in frontend but not in design contract
- `unknown` — cannot confidently classify

Matching priority:
1. `route-map.json` entry `id`
2. Design `screenId`
3. `implementationRoute`
4. Stable screen name
5. Existing `src/design/source-map.json`
6. Agent judgment

## Verification Scripts

Detected from `package.json` scripts:

```text
codegen
lint
test
build
```

Run in order. If a package manager lock file exists, use the corresponding tool:
- `pnpm-lock.yaml` → `pnpm`
- `package-lock.json` → `npm`
- `yarn.lock` → `yarn`
- No lock file → `pnpm` (default)

Always run `pnpm install` first if `node_modules/` is missing.

## Output Files

```text
workspace/design-context/<project>/handoff/design-frontend-update-plan.md
workspace/design-context/<project>/handoff/design-frontend-update-report.md
```

## Prerequisites

This command must run after `/speckit.open-design.process`. Visual comparison is executed separately by `opencode-environment`.

## Error Messaging

If design artifacts are missing, direct the user to run:
```text
./bin/oe speckit:visual <project> all
/speckit.open-design.process --project <project>
```

If the frontend repository is missing, tell the user this command updates existing repositories only and suggest creating frontend staging first.
