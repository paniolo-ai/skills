---
source-slug: typescript-bun-script-portability
source-hash: df745d26258314ebee9414670917633919eac12a910ee163f45ccbd20104bf01
bundled: 2026-08-20
title: TypeScript Bun Script Portability
type: concept
tags:
- authoring
- typescript
- scripts
- bun
- portability
updated: 2026-08-20
---

# TypeScript Bun Script Portability

Use strict TypeScript Bun scripts for cross-platform repository orchestration. Keep executable
`.bun.ts` files thin and move parsing, environment construction, subprocess execution, and cleanup
into importable `.ts` modules with focused tests.

## Subprocess rules

- Pass an argv array to the subprocess API; do not build a shell-quoted command string.
- Set the working directory explicitly.
- Build the child environment explicitly from the parent plus intentional overrides.
- Support setting literal values, copying named parent variables, and unsetting inherited variables
  without shell-specific syntax.
- Capture exit status, stdout, and stderr with typed errors when callers need diagnostics.
- Use an explicit runtime executable for nested Bun scripts.

Avoid `sh -c`, `bash -c`, `cmd /c`, PowerShell command strings, command substitution, and inline
environment-assignment syntax in portable package scripts. These create different quoting and
variable-expansion behavior across PowerShell, WSL, and CI.

When a host-specific system tool is required, invoke its executable with direct argv too. For
example, a Windows process-discovery adapter may start PowerShell with an argument array that names
the script file or encoded operation. Do not fall back to a shell-concatenated command string. Keep
host detection and output parsing behind a typed interface, and return a typed unsupported-host or
tool-unavailable error when no safe adapter exists.

Long-lived child processes need explicit ownership: signal handling, PID tracking, readiness,
failure output, and exact process-tree shutdown. Short-lived commands still need their exit status
propagated unchanged.

## Validation

Test argument parsing and environment transformations as pure functions. Run the real entrypoint on
every supported host family when it performs host-sensitive process management.

## See also

- Entry Point vs Logic Module Pattern
- Effect Subprocesses
- Native Runner Process Ownership
- TypeScript Script Best Practices
