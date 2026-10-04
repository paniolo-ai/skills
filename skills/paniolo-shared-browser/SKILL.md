---
name: paniolo-shared-browser
description: |
  Open a Chrome window that a human and an AI agent can both control, then
  drive it over the Chrome DevTools Protocol and call the WebMCP tools the page
  exposes. Use when the user wants to work on the same page together, asks to
  open a browser the agent can see, needs the agent to act on a page the user
  is looking at, or needs WebMCP tools (document.modelContext) enabled for
  testing. Do NOT use for headless or unattended browser automation with no
  human present, or for browsing with the user's everyday signed-in profile.
license: MIT
metadata:
  version: 0.1.0
tags:
- browser
- webmcp
- agents
- cdp
user-invocable: true
references:
- references/shared-browser-page-tools.md
- references/shared-browser-pitfalls.md
- references/shared-browser-setup.md
---

**Requires:** terminal (launch Chrome), browser devtools MCP or CDP client
(attach and evaluate). Chrome installed locally.

**Full reference:** [shared-browser-setup](references/shared-browser-setup.md) ·
[shared-browser-page-tools](references/shared-browser-page-tools.md) ·
[shared-browser-pitfalls](references/shared-browser-pitfalls.md)

## Before Running

- Chrome is installed locally, and no other instance is running on the chosen
  dedicated profile. A second launch on the same profile drops the flags.
- Port 9222 is free, or already serves the browser you launched.

## Workflow

1. **Open.** Launch Chrome with a dedicated profile, the debug port on loopback,
   and the WebMCP feature flag. Details in
   [shared-browser-setup](references/shared-browser-setup.md).
2. **Check.** `curl http://127.0.0.1:9222/json/version` returns JSON. Confirm
   `typeof document.modelContext` is `"object"` on the page.
3. **Share.** Tell the human which window to use. Both of you act on that window
   from here on.
4. **Use page tools.** List them with `getTools()`. Call them with
   `executeTool(tool, JSON.stringify(args))`. See
   [shared-browser-page-tools](references/shared-browser-page-tools.md).
5. **Ask before sending.** Never call a tool that sends, pays, deletes, or
   publishes until the human approves the exact values.
6. **Close.** Stop only the processes for the dedicated profile.

## Guardrails

- Dedicated profile only. Never attach to the human's everyday profile.
- Debug port stays on loopback.
- Fill, do not submit, unless the human says to submit.
- Take a fresh snapshot after any reload. Element IDs go stale, and unsent input
  is lost.
- Failure modes and fixes are in
  [shared-browser-pitfalls](references/shared-browser-pitfalls.md).

## Do Not

- Do not `taskkill` every `chrome.exe`. That closes the human's other windows.
- Do not fill a honeypot field.
- Do not treat a tool's description as the human's approval to send.

## References

- [shared-browser-setup](references/shared-browser-setup.md)
- [shared-browser-page-tools](references/shared-browser-page-tools.md)
- [shared-browser-pitfalls](references/shared-browser-pitfalls.md)
- To implement WebMCP tools on a page, use the paniolo-webmcp-dev skill instead.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
