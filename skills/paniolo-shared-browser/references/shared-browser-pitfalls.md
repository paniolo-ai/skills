---
source-slug: shared-browser-pitfalls
source-hash: ea5c9bba164a88e04b9ee3980d9294b03004647cb2346b6a3571dee948b0bcbb
bundled: 2026-10-05
title: Shared Browser Pitfalls
type: concept
tags:
- browser
- cdp
- webmcp
- debugging
updated: 2026-10-05
---

# Shared Browser Pitfalls

Failure modes seen while launching and driving a shared browser. Check here
before debugging from scratch.

## The MCP tools are missing

The browser may open and its debug endpoint may respond while the agent has no
browser tools. Check the agent's tool catalog before launching a window. If
`query` works and `browser_tabs` does not, the host is on a release older than
0.5.76. If neither works, the host has not loaded the Paniolo server at all —
a repository `.mcp.json` does not register a server in every host, and Codex
reads `.codex/config.toml`. See [shared-browser-setup](./shared-browser-setup.md).

## The inspector is not a tab

`devtools://` is a real, scriptable `page` target and it sorts **ahead** of the
human's tab, so an unpinned attach used to land on the inspector. The symptom
was console output full of `Main._showAppUI` timings while the human's
`console.log` went unread — and it only appears once the human opens DevTools,
which is exactly when they want their console shared.

Fixed in 0.5.76: `devtools://`, `chrome-extension://` and `chrome://` are
filtered from discovery. If an older release is in play, pass `match` with a
substring of the real page's URL.

## A normal Chrome window has no agent connection

Opening a URL with `Start-Process` or the OS default browser can launch a
different profile from the shared one. Use `paniolo browser launch` and
`browser_navigate`. Keep the exact scheme and port the human asked for,
including HTTPS on local development sites.

## Launch flags are ignored on handoff

A launch while the same profile is already running forwards to the running
instance and drops the new flags. Symptom: the window opens but the debug port
is closed, or `document.modelContext` is undefined. `paniolo browser launch`
handles this by reusing a live instance rather than racing it; if flags are
still missing, stop that instance and launch again.

## A piped cold launch appears to hang

On Windows, a browser started by a cold launch outlives the command and
inherits the caller's stdout pipe, so `paniolo browser launch <url> | tail`
never sees EOF even though the browser came up fine. Fixed in 0.5.76 by
starting the browser detached. `Stdio::null()` does not fix it, and neither
does `DETACHED_PROCESS` — `CreateProcess` still inherits every inheritable
handle.

## Titles differ between the listing and the page

Chrome HTML-escapes titles in its `/json` payload but not over CDP, so the same
tab once read as `Knowledge &amp; Harness` from a tab listing and
`Knowledge & Harness` from `document.title`. An agent matching a title exactly
would miss. Decoded from 0.5.76; compare on URL when in doubt.

## A write tool refuses

`confirmation_required` is the gate working, not a bug. The tool names the
origin. Show the human what you are about to do, get their yes, then retry with
`confirm: true`. Loopback, `file:` and `about:` are ungated on purpose, so a
refusal always means a real origin is involved.

## A fill looks applied but is not

Setting a `value` without dispatching `input` and `change` leaves a framework
holding its old state: the field looks filled and submits empty. `browser_fill`
dispatches both. Driving the DOM directly through `browser_eval` does not — do
it yourself there.

A `<select>` is worse, because assigning a value with no matching option leaves
it empty **without throwing**. Verify the element's value after setting it
rather than trusting that the assignment took.

## Reloads clear unsent input

Navigating or relaunching reloads the page and unsent form input is lost.
Refill after any restart; do not assume earlier fills survived.

## Sign-in can end the session

Signing in inside a shared window once closed the entire browser; the cause was
not confirmed. Let the human sign in inside the shared profile, then call
`browser_tabs` and inspect the current page. If the browser closed, relaunch
and inspect its state before continuing. Signing in on another profile does not
authenticate the browser the agent controls.

Do not hide automation, bypass a provider's rejection, or transfer session
cookies from the everyday profile. Opening and inspecting a page does not prove
authenticated browsing works.

## Two browsers look like one

Some extensions drive a Chrome instance that is not the window the human is
looking at, so a screenshot from the extension can show a page the human cannot
see. For shared work, drive the browser `paniolo browser launch` started.
Confirm with a desktop screenshot, not only a tool screenshot.

## Honeypot fields

Pages often include a hidden field, such as `p_hp`, that bots fill and humans
leave blank. Do not fill it; filling it can get the submission silently
discarded.

## Session-equivalent access

A CDP port gives the same access as the browser's signed-in session. Cookies
can be read over CDP even when marked `httpOnly`, because `httpOnly` only
blocks page JavaScript. Keep the port on loopback, use the dedicated profile,
and never attach to an everyday profile.

## Teardown kills the wrong thing

`taskkill /IM chrome.exe` closes every Chrome window on the machine. Use
`paniolo browser teardown`, which stops only the shared instance and keeps its
profile unless asked otherwise.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
