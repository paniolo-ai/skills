---
name: paniolo-secrets
description: |
  Use the public paniolo secrets CLI to store, check, migrate, rotate, or inject project secrets into a child process. Applies to service names, names-only lists, local credential stores, CI variables, and Windows/WSL vault behavior. Do not use for paniolo-internal Cloudflare secrets operations.
license: MIT
metadata:
  version: 0.1.0
tags:
- secrets
- cli
- keyring
references:
- references/secrets-migration.md
- references/secrets-paniolo-cli.md
---

# Paniolo Secrets

Use the public `paniolo secrets` command for project-scoped secrets. Read
[secrets-paniolo-cli](references/secrets-paniolo-cli.md) for the exact command
contract and [secrets-migration](references/secrets-migration.md) when moving
from an existing keyring or project wrapper.

## Before Running

- Read the project's names-only list and choose its environment-specific
  `--service`. The same name under another service is a different value.
- Keep values out of arguments, files, shell history, logs, and agent output.
  A committed list contains names only.
- Identify whether the project still needs wrapper behavior beyond secret
  injection, such as configuration merging or a fixed working directory.

## Command Workflow

1. Store a human-entered value with `paniolo secrets set NAME --service SERVICE`
   so the prompt hides it. Use `--stdin` for a secure pipeline, and
   `--generate` only when creating a new random value that need not be
   preserved.
1. Check local vault presence with
   `paniolo secrets status --service SERVICE --list FILE`. It reports names
   and set/missing state, never values.
1. Run the child with
   `paniolo secrets run --service SERVICE --list FILE -- COMMAND [ARGS...]`.
   Keep the `--` separator so child flags are passed to the child.
1. Keep the default scrubbed child environment. Use `--inherit-env` only
   when the child requires the rest of the parent environment.
1. For deletion or rotation, confirm the target service and name. A generated
   value cannot be recovered from CLI output; rotating a signing key can
   invalidate existing data or sessions.

## Resolution And Platform Rules

- `run` uses a nonempty listed parent environment value before the vault.
  Locally, any remaining listed names must exist in the selected service or
  the child will not start.
- With nonempty `CI` or `GITHUB_ACTIONS`, `run` skips the local vault.
  Supply listed names through the CI secret mechanism and verify the child
  handles missing values. `status` remains a vault check, not a CI check.
- Windows and WSL reach different credential stores. Use the Windows
  `paniolo.exe` from WSL to access Windows Credential Manager; use
  `--allow-linux-store` only for an intentionally separate Linux vault.
- Python `keyring` and the Rust store use different Windows target names.
  Copy existing values through `set --stdin`, then verify the child command.
  Do not replace a preserved token with `--generate`.
- The public command does not include `paniolo-internal secrets` Cloudflare
  Worker guards or BardoShare's wrapper behavior. Preserve project-specific
  deployment safeguards and environment setup.

## Output And Verification

Report the service, list path, names checked, child exit status, and any
missing names. Never include values or a command that exposes them. For a
migration, verify a representative child operation before retiring old
entries. If a value is unavailable, report the missing name and stop; do not
invent, print, or silently replace it.

## Do Not

- Do not ask for or display a stored value; the CLI has no show command.
- Do not put a secret on the command line or in a names-only list.
- Do not treat a Python keyring entry as proof the Rust store has that entry.
- Do not assume `--inherit-env` is harmless; it forwards unrelated
  credentials.
- Do not assume the public command applies internal production Worker policy.

## References

- [secrets-paniolo-cli](references/secrets-paniolo-cli.md) — commands, lists,
  precedence, CI, and child environment
- [secrets-migration](references/secrets-migration.md) — keyring migration,
  WSL vaults, wrappers, and public/internal boundaries

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
