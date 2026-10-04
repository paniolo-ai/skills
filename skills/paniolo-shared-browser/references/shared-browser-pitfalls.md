---
source-slug: shared-browser-pitfalls
source-hash: 69362f96c8eca0b1fad2bcd8bee4555fd3caddd15638aeed0cd9902387e0df98
bundled: 2026-10-03
title: Shared Browser Pitfalls
type: concept
tags:
- browser
- cdp
- webmcp
- debugging
updated: 2026-10-03
---

# Shared Browser Pitfalls

Failure modes seen while launching and driving a shared browser. Check here
before debugging from scratch.

## MCP Tools Are Missing

The browser may open and its debug endpoint may respond while the agent has
no browser tools. Check the agent's actual tool catalog before launching a
window. A project `.mcp.json` does not register a server in every host:
Codex needs an equivalent `[mcp_servers.chrome-devtools]` entry in its TOML
configuration. Reload the host's MCP connection and verify `list_pages`,
`navigate_page`, and `take_snapshot`. See [shared-browser-setup](./shared-browser-setup.md).

If tools are still absent, inspect host startup errors and whether it can run
`npx`. Do not infer a broken Node installation from a restricted agent shell
alone; check the environment where the host starts MCP servers.

## A Normal Chrome Window Has No Agent Connection

Opening a URL with `Start-Process` or the OS's default browser can launch a
different profile from the MCP server's browser. Use MCP page tools to open
URLs after `list_pages` connects. Keep the exact scheme and port the human
requested, including HTTPS on local development sites.

## The Configured Debug Port Is Closed

With `--browserUrl`, Chrome DevTools MCP only attaches; it does not launch the
browser. Either start the dedicated browser with the configured debug port,
or remove `--browserUrl` and let the server own the launch. Do not mix these
two modes during a task.

## The Site Does Not Expose WebMCP

Use snapshots and normal Chrome MCP click, fill, keyboard, and navigation
tools. A site's missing `document.modelContext` does not block browser control.

## Two browsers look like one

Some extensions drive a Chrome instance that is not the window the human is
looking at. A screenshot from the extension can show a page the human cannot
see. For shared work, drive the browser that exposes the CDP port (see
[shared-browser-setup](./shared-browser-setup.md)). Confirm by taking a desktop screenshot, not only a
tool screenshot.

## Launch flags are ignored on handoff

A launch while the same profile is already running forwards to the running
instance and drops the new flags. Symptom: the window opens, but the debug port
is closed or `document.modelContext` is undefined. Fix: close the running
instance on that profile, then launch again.

## Stale element IDs

Element IDs from a snapshot time out when the page re-renders, or when an
embedded iframe such as a bot-check widget is present. Symptom:
`The element did not become interactive within the configured timeout`, even
though the field exists. Fix: take a fresh snapshot. If it still fails, set the
field through the DOM with the native value setter and dispatch `input` and
`change` events.

## Reloads clear unsent input

Relaunching or navigating reloads the page, and unsent form input is lost.
Refill after any restart, and do not assume earlier fills survived.

## Sign-in can end the session

Signing in inside a shared browser window once closed the entire browser. The
cause was not confirmed. Let the human sign in inside the shared profile,
then call `list_pages` and inspect the current page. If the browser closed,
reconnect the shared profile and inspect its state before continuing. Signing
in on another profile does not authenticate the browser the agent controls.

## Google Rejects The Browser As Insecure

Server-launched Chrome uses automation mode. Some accounts reject sign-in
there with "This browser or app may not be secure." Start Chrome normally
on the dedicated shared profile and configure MCP to attach with
`--browserUrl`, as described in [shared-browser-setup](./shared-browser-setup.md). Reload the host's
MCP connection after changing its arguments. Let the human retry sign-in.
Do not hide automation indicators, bypass Google's rejection, or transfer
session cookies from the everyday profile. Opening and inspecting a page
does not prove authenticated browsing works.

## Tool-call argument format

`executeTool` takes the tool object from `getTools()` and a JSON string of
arguments. See [shared-browser-page-tools](./shared-browser-page-tools.md) for the exact calls and the
errors each mistake produces.

## Honeypot fields

Pages often include a hidden field, such as `p_hp` or a similarly named input,
that bots fill and humans leave blank. Do not fill it. Filling it can get the
submission silently discarded.

## Session-equivalent access

A CDP port gives the same access as the browser's signed-in session. Cookies
can be read over CDP even when marked httpOnly, because httpOnly only blocks
page JavaScript. Keep the port on loopback, use a dedicated profile, and never
attach to an everyday profile.

## Teardown kills the wrong thing

`taskkill /IM chrome.exe` closes every Chrome window on the machine, including
other work. Stop only the processes whose command line includes the dedicated
profile path. See [shared-browser-setup](./shared-browser-setup.md).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
