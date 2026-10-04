---
source-slug: shared-browser-setup
source-hash: 258d92b8497557b7310c74f47e079f177a6b68c0dfb6b6976689f5cc18560a32
bundled: 2026-10-03
title: Shared Browser Setup
type: concept
tags:
- browser
- cdp
- webmcp
- agents
- setup
updated: 2026-10-03
---

# Shared Browser Setup

A shared browser is one Chrome window that a human and an agent both control.
The human uses the mouse and keyboard. The agent attaches over the Chrome
DevTools Protocol (CDP) and sees the same DOM, console, and session. Use it
when the human wants the agent to look at, or act on, the page they are
already viewing.

## Launch

Start Chrome with three things:

1. `--remote-debugging-port=9222` serves CDP on loopback.
2. `--user-data-dir=<dedicated dir>` gives the browser its own profile.
3. `--enable-features=WebMCPTesting` exposes `document.modelContext` (WebMCP)
   on Chrome builds that gate it behind this flag.

Example on Windows (PowerShell):

    Start-Process "C:\Program Files\Google\Chrome\Application\chrome.exe" `
      -ArgumentList "--remote-debugging-port=9222", `
        "--user-data-dir=$env:USERPROFILE\.shared-browser", `
        "--no-first-run", "--enable-features=WebMCPTesting", `
        "https://example.com"

On macOS the binary is
`/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`. On Linux it is
usually `google-chrome` or `chromium` on the PATH. These two were not run
during authoring. Check the path on your machine before you rely on it.

## Rules for the profile

- **Always use a dedicated `--user-data-dir`.** Chrome ignores the debug port
  on the default profile, and attaching to an everyday profile exposes every
  signed-in session to any CDP client.
- **Keep the port on loopback.** Do not bind `--remote-debugging-address` to a
  network interface. Anyone who can reach the port has the browser's session.

## Hand-off: two launches share one profile

Chrome forwards a second launch to the instance already running on the same
profile, and the new flags are dropped. If a shared browser is already open,
close it before launching again. Otherwise the new window opens with no debug
port and no WebMCP flag. Confirm with the check below.

## Check

    curl http://127.0.0.1:9222/json/version

A JSON body with a `webSocketDebuggerUrl` means CDP is up. In a page context,
`typeof document.modelContext` should be `"object"`. If it is `"undefined"`,
the flag is missing or the instance is not the one you launched.

## Teardown

Close the window, or stop only the processes whose command line contains the
dedicated profile path. Do not kill every `chrome.exe`: that closes the
human's other windows and can lose unsaved work.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
