# Spec Kit Design Frontend

A [GitHub Spec Kit](https://github.com/github/spec-kit) extension that applies processed design artifacts to existing frontend repositories.

Spec Kit Design Frontend does **not** create frontend staging, clone UI blueprints, process design exports, or run visual comparison. It updates existing frontend code to match the latest design contract while preserving production application behavior.

## What this extension does

Spec Kit Design Frontend adds the command:

```text
/speckit.design-frontend.update
```

The command updates an existing frontend repository by:

* loading the processed design contract (IR, tokens, component contracts, page structures, data mappings)
* inspecting the current frontend codebase
* classifying screens as unchanged, changed, new, or removed-from-design
* generating a design-to-frontend update plan
* applying targeted edits to changed screens
* creating routes, components, and static fixtures for new screens
* updating route-map.json implementationRoute values for new screens
* copying design reference files into the frontend `src/design/` directory
* preserving production behavior (auth, routing, GraphQL integration, business logic)
* running verification scripts (codegen, lint, test, build)

## What this extension does NOT do

* Create `frontend-staging` from a UI blueprint
* Clone `ui-apollo-blueprint`
* Crawl Open Design exports
* Generate screenshots
* Process raw design exports (that belongs to `/speckit.open-design.process`)
* Run visual comparison (that belongs to `opencode-environment`)
* Delete production screens automatically
* Replace or reset production frontend repositories

## Requirements

* GitHub Spec Kit
* `specify` CLI with extension support
* Spec Kit version `>=0.8.14`
* Processed design artifacts (run `/speckit.open-design.process` first)
* An existing frontend repository under `workspace/frontend/<frontend-repo>`

If you have not installed Spec Kit yet, follow the official Spec Kit installation instructions:

```bash
uvx --from git+https://github.com/github/spec-kit.git specify init my-project
```

Or install the CLI as a tool:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

## Installation

### Install from this repository

From your Spec Kit project directory, clone this repository and add it as an extension:

```bash
git clone https://github.com/reytech-dev/spec-kit-design-frontend.git
specify extension add --dev ./spec-kit-design-frontend
```

Verify that the extension was installed:

```bash
specify extension list
```

You should see the `Specify Design Frontend` extension listed with its provided command.

### Install from a local checkout

If you already have this repository checked out locally:

```bash
cd /path/to/your/spec-kit-project
specify extension add --dev /path/to/spec-kit-design-frontend
```

## Usage

Start your AI coding agent in your Spec Kit project and run:

```text
/speckit.design-frontend.update --project workbench --frontend-repo workbench
```

### Required flags

* `--project <slug>` — project slug matching `workspace/design-context/<project>`
* `--frontend-repo <repo>` — frontend repository name matching `workspace/frontend/<frontend-repo>`

### Optional flags

| Flag | Default | Description |
|---|---|---|
| `--dry-run` | `false` | Analyze inputs and produce an update plan only |
| `--static-data` | `true` | Use static GraphQL fixtures for deterministic visual states |
| `--no-static-data` | — | Do not create or update static GraphQL fixtures |
| `--preserve-production-behavior` | `true` | Preserve existing real application behavior, auth, routing, and GraphQL integration |
| `--allow-package-changes` | `false` | Permit `package.json` changes if required |
| `--update-route-map` | `true` | Fill missing `implementationRoute` values for new screens |
| `--no-update-route-map` | — | Do not modify `route-map.json` |
| `--visual-target` | `true` | Optimize changes toward passing visual comparison |
| `--max-files <number>` | none | Safety cap for number of files to modify |
| `--help` | — | Show usage |

### Dry run

To preview changes without modifying files:

```text
/speckit.design-frontend.update --project workbench --frontend-repo workbench --dry-run
```

This produces a plan at `workspace/design-context/<project>/handoff/design-frontend-update-plan.md`.

### Example workflow

```bash
# 1. Capture visual design (outside the agent)
./bin/oe speckit:visual workbench all

# 2. Process design artifacts (inside the agent)
/speckit.open-design.process --project workbench

# 3. Preview the update plan
/speckit.design-frontend.update --project workbench --frontend-repo workbench --dry-run

# 4. Apply the update
/speckit.design-frontend.update --project workbench --frontend-repo workbench

# 5. Start the updated frontend (outside the agent)
./bin/oe frontend:start workbench

# 6. Run visual comparison (outside the agent)
./bin/oe speckit:visual workbench compare --frontend-url http://node-runner:5173 --no-fail-on-diff
```

## Command behavior

The update command follows these rules:

* It must run **after** `/speckit.open-design.process`.
* It validates that all required design artifacts exist before modifying anything.
* It validates that the frontend repository exists.
* It classifies every design screen as unchanged, changed, new, or removed-from-design.
* It generates an update plan and writes it to `handoff/design-frontend-update-plan.md`.
* With `--dry-run`, it stops after the plan and does not modify files.
* For changed screens, it makes targeted edits to existing files.
* For new screens, it creates routes, components, and fixtures following existing frontend conventions.
* Removed-from-design screens are reported but **never** automatically deleted.
* Production behavior (auth, routing, GraphQL integration, business logic) is preserved.
* Static fixtures are only added or updated for deterministic visual states.
* Design reference screenshots are never modified.
* `frontend-staging/` is never created or modified.
* `ui-apollo-blueprint` is never cloned.
* After modifications, it runs codegen, lint, test, and build scripts when available.
* It writes a final report to `handoff/design-frontend-update-report.md`.

## Output files

The command produces two output files:

```text
workspace/design-context/<project>/handoff/design-frontend-update-plan.md
workspace/design-context/<project>/handoff/design-frontend-update-report.md
```

## Development

Clone the repository:

```bash
git clone https://github.com/reytech-dev/spec-kit-design-frontend.git
cd spec-kit-design-frontend
```

Install it into a test Spec Kit project:

```bash
cd /path/to/spec-kit-project
specify extension add --dev /path/to/spec-kit-design-frontend
```

Check that the command was registered:

```bash
specify extension list
```

For AI agent integrations, the command should be available as:

```text
/speckit.design-frontend.update
```

## Uninstall

Remove the extension from a Spec Kit project:

```bash
specify extension remove design-frontend
```

## Versioning

This extension follows semantic versioning.

Commit messages should follow Conventional Commit style:

```text
<type>(<scope>): <subject>
```

Allowed types:

```text
feat, fix, test, refactor, docs, perf, chore
```

Examples:

```text
feat(update): add screen classification logic
fix(verify): handle missing pnpm gracefully
docs(readme): document --dry-run behavior
chore(release): configure semantic release
```

## License

MIT
