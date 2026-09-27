---
source-slug: secrets-paniolo-cli
source-hash: f33c815b54cd7818d5615438c5ee81055b52ebbec502a207ffc2e42a0db79028
bundled: 2026-09-26
title: Paniolo Secrets CLI
type: concept
tags:
- secrets
- cli
- keyring
- security
- agents
updated: 2026-09-26
---

# Paniolo Secrets CLI

The public `paniolo secrets` command stores project secrets in the operating
system credential store and injects selected names into one child process.
Use a project-owned service name and a committed list of names. Keep values
out of files, command arguments, shell history, and agent output.

## Contents

- [Command Contract](#command-contract)
- [List Files and Service Names](#list-files-and-service-names)
- [Run Resolution and Child Environment](#run-resolution-and-child-environment)
- [Safe Agent Workflow](#safe-agent-workflow)
- [See Also](#see-also)

---

<a id="command-contract"></a>

## Command Contract

| Action | Command | Behavior |
| --- | --- | --- |
| Store | `paniolo secrets set NAME --service SERVICE` | Prompts without echo; replaces an existing value. |
| Store from pipe | `paniolo secrets set NAME --service SERVICE --stdin` | Reads one value from stdin; strips trailing CR/LF. |
| Generate | `paniolo secrets set NAME --service SERVICE --generate` | Stores a random 256-bit base64 value without displaying it; refuses overwrite. |
| Check | `paniolo secrets status --service SERVICE --list FILE` | Prints names and set/missing status, never values; exits nonzero if any are missing. |
| Run | `paniolo secrets run --service SERVICE --list FILE -- COMMAND [ARGS...]` | Runs a child with listed secrets; passes through its output and exit status. |
| Delete | `paniolo secrets delete NAME --service SERVICE` | Removes one entry; reports absence without failing. |

There is no command to print a stored value. `status` reads the credential store;
it does not count a value supplied only through the current shell environment.
`set --stdin` and `set --generate` cannot be combined. A generated secret cannot
be recovered from CLI output. To rotate one deliberately, delete it and generate
a new entry as separate actions.

---

<a id="list-files-and-service-names"></a>

## List Files and Service Names

The project supplies `--service` for every action. Use different service names
for development, staging, and production. A matching variable name under two
services does not imply a shared value.

`status` and `run` require `--list`. The list is plain text with one portable
environment variable name per line. Blank lines and `#` comments are ignored;
duplicate names and malformed names fail. Never put a value in the list.

```text
# config/env-secrets.dev.list
DATABASE_URL
API_TOKEN # token used by the child
```

A name starts with an ASCII letter or underscore and continues with ASCII
letters, digits, or underscores. This makes the same list usable on Windows
and Unix. The list may be committed because it contains names only.

---

<a id="run-resolution-and-child-environment"></a>

## Run Resolution and Child Environment

For each listed name, `run` uses a nonempty parent environment value first.
Locally, it reads a missing value from the selected credential-store service
and fails before starting the child if any name remains missing. Four or more
vault reads use at most four worker threads; smaller sets are read serially.

When `CI` or `GITHUB_ACTIONS` has a nonempty value, `run` never queries the
local vault. It passes populated listed variables and leaves unresolved names
for the child to handle. Test the child's own missing-variable behavior in CI;
`status` is a separate vault check and is not the CI readiness check.
The CI marker itself is scrubbed from the child unless it is listed or full
environment inheritance is enabled.

By default, the child receives only essential operating-system variables and
the listed secrets. Unrelated tokens in the agent's shell are removed. Pass
`--inherit-env` only when the child genuinely needs the full parent
environment; that option also passes unrelated credentials. Secret values
exist in process memory and are not written to an env file by this command.

Put the child command after `--` so its flags remain child arguments. On
Windows, a missing bare executable can be retried as a `.cmd` shim without
passing arguments through `cmd /c`. The child writes directly to the terminal.
A normal child exit code becomes the CLI exit code; a signal maps to 130.

---

<a id="safe-agent-workflow"></a>

## Safe Agent Workflow

1. Read the project's secret list and service naming convention. Keep names
   visible, but never ask an agent to reveal a stored value.
1. For a human-entered value, use `set` and its hidden prompt. For a secure
   pipeline, use `--stdin`. Do not put the value on the command line.
1. Use `status` to check the local vault without printing values.
1. Run the project's command with `secrets run`. Prefer the scrubbed child
   environment and add `--inherit-env` only for a documented dependency.
1. For CI, inject listed environment names through the CI secret mechanism and
   verify the child fails clearly when a required name is absent.
1. Rotate at the source of truth before updating any deployed copy. Do not
   accidentally replace an unrecoverable generated signing key.

For platform vault differences and existing project wrappers, follow
[secrets-migration](./secrets-migration.md). The Paniolo-specific internal CLI also has Cloudflare
operations that are outside this public command.

---

<a id="see-also"></a>

## See Also

- Migrating to Paniolo Secrets
- Secrets Management
- Cross-platform keyring notes
