---
source-slug: webmcp-timeline
source-hash: c65bdf4149dd2940679102c025b658c25101644a936455ecbf0b8c74e6fb54cc
bundled: 2026-09-25
title: WebMCP Rollout Timeline
type: concept
tags:
- webmcp
- browser
- chrome
- mobile
- timeline
updated: 2026-09-25
---

# WebMCP Rollout Timeline

Estimated browser and consumer rollout as of **2026-09-25**. Chromium milestones
come from the Blink [Intent to Experiment](https://groups.google.com/a/chromium.org/g/blink-dev/c/gmYffo5WOE8/m/OJxuQRP3AAAJ);
calendar dates map those milestones to Chromium Dash `early_stable` (Stable
launch estimates — not ship commitments). Non-Chromium engines remain uncommitted.

## Chromium (Chrome / Edge / WebView)

Intent milestones apply to **desktop, Android, and Android WebView** the same
way (all six Blink platforms listed as yes).

| Phase | Milestone | Est. Stable (`early_stable`) | Notes |
| --- | --- | --- | --- |
| DevTrial | 146 | 2026-02-25 | Early local experimentation |
| Origin trial starts | 149 | 2026-05-20 | Public OT; Chrome docs + testing flag |
| API rename era | 150 | 2026-06-17 | `navigator.modelContext` → back-compat; prefer `document.modelContext` |
| Abort / unregister nuance | 153 | 2026-08-26 | Unregister no longer cancels in-flight executes |
| String `executeTool` args deprecated | 155 | 2026-09-23 | Pass serializable objects |
| Origin trial ends | 156 | 2026-10-07 | Last OT milestone (inclusive) |
| **Estimated shipping** | **157** | **2026-10-21** | Desktop, Android, and WebView — Intent estimate only |

Local development today: `chrome://flags/#enable-webmcp-testing` (or
`--enable-features=WebMCPTesting`). OT token alone has not always been enough
without the flag — see [blog-vietanh-webmcp-attack-surface](./blog-vietanh-webmcp-attack-surface.md).

**Edge:** origin trial live from **Edge 150** (Chromium-aligned; defer to Chrome
platform notes for desktop/Android/WebView detail).

## Other browsers

| Engine | Status (2026-09) | Estimate |
| --- | --- | --- |
| Firefox (Gecko) | Standards-position + Bugzilla open; Intent: "No signal" | **No ship date** — discussion only |
| Safari (WebKit) | Standards-position open; Intent: "No signal" | **No ship date** — discussion only |
| Brave | Experimental Leo AI chat support | Experimental; not a platform ship |

Cross-browser interoperable WebMCP is **not** on a calendar. Treat it as
Chromium-first progressive enhancement: `if (document.modelContext) { … }`.

## Spec / standards track

| Track | Status |
| --- | --- |
| W3C WebML CG draft | Incubating (`webmachinelearning.github.io/webmcp`) |
| TAG review | Pending at Intent time — requested before shipping |
| W3C Standards Track | Not on it yet |

## Agent / consumer surfaces (not browsers)

These matter for "will anyone call my tools?" more than engine ship dates:

| Consumer | Status (sources ~Aug–Sep 2026) |
| --- | --- |
| ChatGPT Desktop ("Site tools") | Shipping — imperative, top-level page |
| Chrome extensions / Prompt API | Can discover and call tools during OT |
| Gemini in Chrome | Announced as a consumer; not confirmed shipping in digests |
| Shopify Liquid / Hydrogen | Platform enabled WebMCP tools on storefronts |

## How to read this

- **~Oct 2026 (Chrome 157)** is the Intent's ship *estimate* for Chromium
  desktop, Android, and WebView — watch Chrome Status and a future Intent to
  Ship; OT ends at 156.
- **Mobile** here means Chrome on Android and Android WebView on the same
  milestone train as desktop. iOS Chrome uses WebKit — no WebMCP until WebKit
  moves.
- **Firefox / Safari:** months-to-years unknown; do not plan product-critical
  paths on them.
- Keep integrations thin ([webmcp-overview](./webmcp-overview.md), [webmcp-apis](./webmcp-apis.md)) so milestone
  churn stays cheap.
