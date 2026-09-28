# AGENTS.md — plugin-agentteams

Standalone plugin repo for the AgentTeams controller CLI (`command:agentteams`)
and check verb (`verb:agentteams`). The plugin is a Go module at
`candy/plugin-agentteams/` (module path
`github.com/opencharly/plugin-agentteams/candy/plugin-agentteams`); the root
`charly.yml` only declares `discover: candy` so the repo is a project and its
candy is scanned.

Canonical files:

- `candy/plugin-agentteams/charly.yml` — the `plugin-agentteams:` candy entity
  and the embedded `agentteams-cli-skill:` skill entity.
- `candy/plugin-agentteams/` — the Go source: `plugin.go`, `control.go` (the
  REST client + command tree), `verb.go`, `schema/agentteams.cue`,
  `params/cue_types_gen.go`, `cmd/serve/main.go`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-agentteams:agentteams-cli` — the compiled-in `charly agentteams` CLI
  and the `agentteams:` check verb reference. Load before changing the command
  tree or the verb.
- `/charly-agentteams:agentteams` — the AgentTeams box composition, volumes,
  ports, and deploy substrates the CLI targets.
- `/charly-internals:plugin` — the plugin authoring reference: the `plugin:`
  block, the unified Provider model, the per-plugin CUE-schema contract. Load
  before touching the provider or schema.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `go build ./...` in `candy/plugin-agentteams/` — compile the plugin module.
- `go test ./...` in `candy/plugin-agentteams/` — the plugin's Go tests (the
  REST-client e2e lives in `control_test.go`).
- `charly box validate` at the repo root — the structural check (the candy +
  `plugin:` block, CUE schema).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate.
- The live R10 witness is the `check-agentteams-pod` bed roster (the host steps
  driving `charly agentteams status` / `worker list` against a live controller).

## Modify this repo

- Edit the `plugin-agentteams:` candy entity, the Go source, and
  `schema/agentteams.cue` **together** — the schema is the single source for the
  verb's `params/` struct.
- The CLI and the check verb share ONE REST client (R3); change the client, not
  a copy, when the controller contract moves.
- Keep the `agentteams-cli-skill:` entity in step with any command-tree change —
  it is the projected source for `/charly-agentteams:agentteams-cli`.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
