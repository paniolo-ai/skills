---
source-slug: secrets-migration
source-hash: a499155a0fed0d85d01fe3245406400ce2948a88a54fc395b4fcdd167052802d
bundled: 2026-09-27
title: Migrating to Paniolo Secrets
type: concept
tags:
- secrets
- migration
- keyring
- wsl
- bardoshare
updated: 2026-09-26
---

# Migrating to Paniolo Secrets

Paniolo's public secrets command uses Rust's credential-store client. Moving an
existing project to it requires copying values into that client's entries and
preserving any behavior supplied by the old project wrapper. Migrate one
service and environment at a time; keep the previous entries until the new
runner has been verified.

## Contents

- [Why Values Need Copying](#why-values-need-copying)
- [Migration Sequence](#migration-sequence)
- [BardoShare Wrapper Behavior](#bardoshare-wrapper-behavior)
- [Windows and WSL Vaults](#windows-and-wsl-vaults)
- [Public and Internal Command Boundaries](#public-and-internal-command-boundaries)
- [See Also](#see-also)

---

<a id="why-values-need-copying"></a>

## Why Values Need Copying

Python `keyring` and Rust `keyring` use different Windows Credential Manager
target names. Even when both use the same service and variable name, a value
stored by Python can appear missing to `paniolo secrets status`. Treat that
result as a namespace difference until the migration is checked. Do not
switch a project runner merely because the service strings match.

The public CLI does not display stored values. Copy from the old client
straight into `paniolo secrets set --stdin`. Do not use `--generate` when
preserving an existing token or signing key: it creates a new value.

---

<a id="migration-sequence"></a>

## Migration Sequence

1. Inventory every environment-specific service and names-only list. Keep
   development, staging, and production values separate.
1. For each listed name, pipe the old value into the Rust store. The example
   uses one BardoShare development entry; substitute the actual service and
   name on each pass.
1. Run `status` against the same service and list. It checks vault presence
   without revealing values.
1. Run a representative project command through `paniolo secrets run` and
   verify its behavior and exit status. Repeat in each environment before
   changing any script.
1. Switch the wrapper only after its non-secret behavior and working directory
   are accounted for. Keep old Python entries until the new path is proven.
1. Treat deletion of old entries as a separate cleanup or rotation decision.

```powershell
keyring get bardoshare-dev API_TOKEN |
  paniolo secrets set API_TOKEN --service bardoshare-dev --stdin

paniolo secrets status --service bardoshare-dev --list config/env-secrets.dev.list
paniolo secrets run --service bardoshare-dev --list config/env-secrets.dev.list -- my-command
```

The example's `--list` path is resolved from the current working directory.
Do not print either client's value for comparison. Compare the child command's
observable behavior instead. If a local value is missing, the public runner
fails before launching the child.

---

<a id="bardoshare-wrapper-behavior"></a>

## BardoShare Wrapper Behavior

BardoShare's current `run-with-env` path does more than fetch vault entries.
It merges project configuration, applies set/default/copy/unset environment
overrides, and runs children from the BardoShare project root. The public
`paniolo secrets run` command supplies listed secrets and essential process
variables; it does not perform those project-specific steps.

Keep the BardoShare wrapper or replace it with a thin wrapper that preserves
those steps. Verify each script's environment, working directory, and missing
secret behavior before switching. The Rust migration described here does not
change BardoShare scripts by itself.

---

<a id="windows-and-wsl-vaults"></a>

## Windows and WSL Vaults

A Linux build inside Windows Subsystem for Linux (WSL) reaches a separate
Linux Secret Service vault. `set`, `status`, `delete`, and `run` when a vault
read is needed refuse that vault by default. From WSL, invoke the Windows
`paniolo.exe` through `/mnt/c` to use the same Windows Credential Manager as
PowerShell. Use `--allow-linux-store` only when the project intentionally
maintains a separate WSL-local vault.

A `run` call whose listed values are already nonempty in the environment
does not need a vault read. In CI, `CI` or `GITHUB_ACTIONS` makes the public
runner skip the local vault entirely. These exceptions do not make the two
stores interchangeable.

---

<a id="public-and-internal-command-boundaries"></a>

## Public and Internal Command Boundaries

The public `paniolo secrets` command is project-configured. It stores, checks,
deletes, and injects values. It does not know Paniolo's Cloudflare Worker
names or impose Paniolo's production Worker policy.

`paniolo-internal secrets` retains Paniolo-specific behavior: `push-worker`
streams named keyring values to Wrangler through stdin, sets the Wrangler
process-environment flag for local development, and refuses a local
`wrangler dev` process with a production service. Do not assume those guards
exist in the public command. A project adopting the public runner must keep
its own deployment and production safeguards.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
