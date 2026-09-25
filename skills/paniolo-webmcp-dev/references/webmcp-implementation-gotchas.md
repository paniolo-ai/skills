---
source-slug: webmcp-implementation-gotchas
source-hash: 01a5c28a636bb668c1f87101d6bba91dd2715dd9b4d10a018f88855a47955bb0
bundled: 2026-09-25
title: WebMCP Implementation Gotchas
type: concept
tags:
- webmcp
- debugging
- deployment
- testing
updated: 2026-09-25
---

# WebMCP Implementation Gotchas

Field notes from shipping `document.modelContext.registerTool` on a real
production page (`paniolo.ai` contact form). None of these are documented in
the spec or Chrome's guides, and each one produces **silent failure** — no
thrown error, no console warning, tools simply never appear. Mechanics:
[webmcp-apis](./webmcp-apis.md). Rollout status: [webmcp-timeline](./webmcp-timeline.md).

## 1. `document.modelContext` can attach after your script already ran

A page's `<script>` tag (no `defer`) executes synchronously during HTML
parsing. In at least one real Chromium build (Edge 155 with
`edge://flags/#enable-webmcp-testing`), `document.modelContext` was not yet
present at that exact moment — it showed up shortly after, before the `load`
event, but too late for a **one-shot synchronous feature-detect**:

```js
// Broken: silently no-ops if modelContext attaches even a moment late.
function registerTools() {
  const ctx = document.modelContext;
  if (!ctx) return; // never retried
  ctx.registerTool(/* ... */);
}
registerTools();
```

**Fix: poll briefly before giving up**, not just once:

```js
function waitForModelContext() {
  if (document.modelContext) return Promise.resolve(document.modelContext);
  return new Promise((resolve) => {
    let attempts = 0;
    const timer = setInterval(() => {
      attempts++;
      const ctx = document.modelContext;
      if (ctx || attempts >= 20) { // ~10s at 500ms
        clearInterval(timer);
        resolve(ctx || null);
      }
    }, 500);
  });
}

async function registerTools() {
  const ctx = await waitForModelContext();
  if (!ctx) return;
  await ctx.registerTool(/* ... */);
}
```

To confirm this is really what's happening (rather than a permissions or
schema error being swallowed), reproduce it directly: temporarily hide
`document.modelContext` with `Object.defineProperty`, run your registration
code, then restore it after a delay and confirm your poll picks it up. If a
one-shot check would miss that and your poll doesn't, you've found (and
fixed) the same bug.

## 2. The WebMCP testing flag may require a secure context

`chrome://flags/#enable-webmcp-testing` (or the Edge equivalent) exposed
`document.modelContext` on `https://` origins but left it `undefined` on
plain `http://localhost` in the same browser, same session — even though
`localhost` is normally treated as a secure context for most other
browser APIs. Do not assume your local dev server can exercise WebMCP just
because the flag is on. If you need to test locally, serve over HTTPS (a
local cert, `mkcert`, or a tunnel), or do a final check against a real HTTPS
deployment (staging is fine — see gotcha 3 first).

## 3. A referenced script that isn't actually deployed fails silently as HTML, not a 404

If your build step forgets to copy the tool-registration script into the
deployed output, a static host with SPA-style fallback routing
(`not_found_handling: single-page-application` on Cloudflare Pages, and
equivalents elsewhere) does **not** 404. It serves your app shell / index
page in its place, with **HTTP 200**. Symptoms:

- `document.modelContext.getTools()` stays empty indefinitely, no errors.
- The page otherwise works fine (other scripts loaded correctly), so nothing
  looks broken.
- `fetch('/js/your-script.js')` from the page's own console may *also* be
  routed to the fallback — don't trust that fetch's 200 status either.

**How to actually verify what shipped:**

- Navigate a tab directly to the script's URL. If it renders your site
  instead of showing raw JS text, the asset does not exist server-side.
- Check the real build output directory (`dist/`, or whatever your bundler
  produces) for the file, not just the source tree. A script referenced in
  `index.html` proves nothing about whether the build step copies it.
- Grep your build/prepare script for every asset your HTML references — a
  copy-list that adds files one at a time (`cpSync(a); cpSync(b);`) silently
  drops any file someone forgets to add to the list.

## 4. Caching hides both the bug and the fix

Once the real bug (gotcha 3) is fixed and redeployed, you can still observe
the *old broken behavior* for a while, at two layers:

- **CDN edge cache** — a wrongly-cached fallback response for the script's
  URL can outlive the fix by its full `Cache-Control: max-age`. A
  cache-busting query string forces a fresh edge fetch and tells you whether
  the origin is actually fixed.
- **Browser disk cache** — even a brand-new tab in the same browser profile
  can serve a disk-cached copy of the script (`performance.getEntriesByType
  ('resource')` shows `transferSize: 0` for a cache hit vs a real number for
  a network fetch). A normal reload is not enough; hard-reload
  (Ctrl/Cmd+Shift+R) to bypass it when verifying a deploy.

Verify the fix landed by checking both layers independently before
concluding a deploy did or didn't work.

## 5. Showing a "supports WebMCP" badge: gate on real registration, not just feature-detection

If you show UI chrome announcing WebMCP support, reveal it **after your
tools actually finish registering**, not merely after `document.modelContext`
exists — the point is to prove real capability, not just repeat the
feature-detect as marketing copy:

```js
async function registerTools() {
  const ctx = await waitForModelContext();
  if (!ctx) return;
  await ctx.registerTool(/* ... */);
  document.querySelector('#webmcp-badge').hidden = false; // reveal on success
}
```

Watch the CSS: an element hidden with the `hidden` attribute relies on the
browser's low-specificity `[hidden] { display: none }` rule. Any author rule
that sets `display` directly on that element (`.badge { display:
inline-flex }`) has equal specificity and — because author styles are later
in the cascade than the user-agent stylesheet — **wins**, so the element
shows regardless of the `hidden` attribute. Add an explicit
`.badge[hidden] { display: none }` override whenever you set `display` on an
element that also toggles `hidden`.

## Related

- [webmcp-apis](./webmcp-apis.md) — `registerTool` / `getTools` / `executeTool` mechanics
- [webmcp-best-practices](./webmcp-best-practices.md) — tool design once registration itself works
- [webmcp-timeline](./webmcp-timeline.md) — which browsers/flags expose `document.modelContext` today
- [webmcp-observability](./webmcp-observability.md) — telling registration failures apart from model mistakes in production
