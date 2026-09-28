# plugin-agentteams

The `charly agentteams` management CLI and the declarative `agentteams:` check
verb for OpenCharly — a compiled-in `command:agentteams` + `verb:agentteams`
plugin.

It is a plain `net/http` REST client for the AgentTeams controller: no upstream
`agt` binary, no SDK. Use it to manage a running AgentTeams deployment — create
and inspect Workers, apply Teams and Humans, and check controller health. A
Human is a real person (a human-in-the-loop participant with an identity and
permission tier) who can observe and intervene in the Rooms their permissions
allow.

## What it provides

| Capability | Surface |
|---|---|
| `command:agentteams` | `charly agentteams status`, `worker list\|get\|create\|update\|apply\|delete`, `team apply\|list\|get\|delete`, `human apply\|list\|get\|delete`, `apply -f <file>`, `config` |
| `verb:agentteams` | the `agentteams:` check verb — `status`, `manager-running`, `worker-running`, `worker-list` |

The verb is the controller-probe counterpart of the CLI: it resolves the
controller's in-venue `:8090` to a host-routable address over the reverse
channel, pulls the admin service-account token from the venue, and probes with
the **same** REST client the command uses. It skips under `charly check box`
(no live controller on a disposable `podman run --rm`).

## How to use it

The CLI is compiled in. Point it at a controller:

```bash
export AGENTTEAMS_CONTROLLER_URL=http://127.0.0.1:8090
export AGENTTEAMS_AUTH_TOKEN_FILE=/path/to/cli-token
charly agentteams status
charly agentteams worker list
```

Endpoint comes from `AGENTTEAMS_CONTROLLER_URL` (default
`http://127.0.0.1:8090`); the token from `AGENTTEAMS_AUTH_TOKEN` or
`AGENTTEAMS_AUTH_TOKEN_FILE` (the upstream runtime contract). The controller
mints the admin SA token to `/var/run/agentteams/cli-token`; on a pod substrate
pull it out with `charly cp <deploy> :/var/run/agentteams/cli-token <local>`.

Author the check verb in a plan:

```yaml
- check: the controller is healthy
  agentteams: status
  context: [runtime]
```

## Layout

- `candy/plugin-agentteams/` — the plugin module: `plugin.go`, `control.go`
  (the REST client + command tree), `verb.go`, `s3.go`, `snapshot.go`,
  `schema/agentteams.cue`, `params/cue_types_gen.go`, `cmd/serve/main.go`.
- `candy/plugin-agentteams/charly.yml` — the `plugin-agentteams:` candy entity
  and the embedded `agentteams-cli-skill:` skill entity.
- `charly.yml` — the root project manifest (`discover: candy`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.

## Related

- Owning skill: `/charly-agentteams:agentteams-cli` (projected from the embedded
  `agentteams-cli-skill:` entity).
- `/charly-agentteams:agentteams` — the AgentTeams box: composition, volumes,
  ports, both deploy substrates.
- `/charly-internals:plugin` — the plugin/provider model.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI.
