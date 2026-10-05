---
source-slug: shared-browser-setup
source-hash: 11e9a1fc5698c353d98b45b87cb478aaf57ab30906d38a41c729b85b530a632e
bundled: 2026-10-05
title: Shared Browser Setup
type: concept
tags:
- browser
- cdp
- webmcp
- agents
- setup
updated: 2026-10-05
---

# Shared Browser Setup

A shared browser is one Chrome window that a human and an agent both control.
The human uses the mouse and keyboard. The agent attaches over the Chrome
DevTools Protocol and sees the same DOM, console, and session.

Setup used to be the long part of this: a separate browser MCP server pinned in
`package.json`, plus a config block for every vendor. None of that applies from
`@paniolo/cli` 0.5.76. The `browser_*` tools live on the Paniolo MCP server the
harness already configures, so if `query` works in this host, the browser tools
work too.

## Check first

Call `browser_tabs`. Three outcomes:

| Result | Meaning |
| --- | --- |
| A tab list, or "no open tabs" | Connected. Nothing to set up. |
| `browser_unavailable` | Server loaded, no browser running yet. Launch one. |
| Tool not found | Host has not loaded the Paniolo server, or is pre-0.5.76. |

An unknown-tool error is a host configuration problem, not a browser problem.
Do not launch a window to investigate it.

## Launch

```bash
paniolo browser launch https://example.com
```

That starts a Chromium-family browser with a dedicated profile at
`~/.paniolo-browser`, `--remote-debugging-port=9222`, and
`--enable-features=WebMCPTesting`.

The profile is a boundary, not a convenience: the debug port carries the same
access as the browser's signed-in session, so it must never be the everyday
profile. Chrome also ignores `--remote-debugging-port` on a default profile,
which forces the isolation anyway.

A second launch against a running instance **reuses** it and opens the URL
there. That matters, because launching the same profile twice is how the
debugging flags get silently dropped.

`--port` runs a second instance when one browser genuinely is not enough. One
port per browser is the unit; there is no multiplexing.

`PANIOLO_BROWSER_BIN` picks the executable when the search misses the one you
want. The search covers Chrome and Edge on Windows, Chrome and Chromium on
macOS, and the usual `/usr/bin` and snap paths on Linux.

## Verify

```bash
paniolo browser tabs
```

or `browser_tabs` from the agent. The listing shows only the human's tabs:
`devtools://`, `chrome-extension://` and `chrome://` targets are filtered out,
because the DevTools window is a real scriptable page that would otherwise sort
ahead of the tab the human is looking at.

## Teardown

```bash
paniolo browser teardown
```

Stops the instance over CDP — its own shutdown path, so profile state is
flushed rather than truncated — and keeps the profile. `--wipe-profile`
discards it, sign-ins included. Never `taskkill /IM chrome.exe`; that closes
the human's other windows too.

## Why the write tools ask

`browser_eval`, `browser_navigate`, `browser_click`, `browser_fill`,
`browser_type` and `browser_webmcp_call` refuse on any origin that is not
loopback, `file:` or `about:`, returning `confirmation_required` and naming the
origin. Retry with `confirm: true` once the human agrees.

This exists because the port is session-equivalent access, so "evaluate this
expression" against a signed-in production tab is a different act from the same
expression against a dev server. Loopback is ungated on purpose: a flag
everyone always passes has stopped being a gate.

Reads — `browser_tabs`, `browser_console`, `browser_screenshot`,
`browser_webmcp_list` — are never gated.

## What this replaced

`chrome-devtools-mcp` was removed from the harness in favour of these tools.
Four capability families are deliberately not replaced, because nothing in them
can act as the user: Lighthouse audits, performance traces, heap snapshots, and
PWA or extension tooling. If one of those is load-bearing for a task, add that
server for that task rather than keeping it loaded always.

Two workflow differences are worth knowing up front:

- **No snapshot-and-click-by-id.** Interaction is by CSS selector; use
  `browser_eval` to discover selectors on an unfamiliar page.
- **No new-tab tool.** `paniolo browser launch <url>` opens a URL in the
  running instance, and `browser_navigate` moves an existing tab.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
