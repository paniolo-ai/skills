---
source-slug: shared-browser-pitfalls
source-hash: 9ef478580d0d38a165f19553313346125f13364aee9082468bd9dfa8f9a1452e
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
cause was not confirmed. Sign in on a separate profile, or close the shared
browser before signing in elsewhere, and verify the port afterwards.

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
