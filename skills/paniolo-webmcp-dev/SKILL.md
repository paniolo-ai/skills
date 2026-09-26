---
name: paniolo-webmcp-dev
description: |
  Implement, debug, test, or review WebMCP (document.modelContext) tools that expose a page's client-side actions to browser AI agents. Use when adding registerTool/declarative form tools, deciding WebMCP vs an MCP server, writing tool descriptions/schemas, handling cross-origin or security annotations, or diagnosing "tools never register" / "modelContext is undefined" bugs. Do NOT use for a backend MCP server with no browser page involved — that is plain MCP, not WebMCP.
license: MIT
metadata:
  version: 0.1.0
tags:
- webmcp
- browser
- agents
references:
- references/webmcp-apis.md
- references/webmcp-best-practices.md
- references/webmcp-cross-origin.md
- references/webmcp-evals.md
- references/webmcp-implementation-gotchas.md
- references/webmcp-observability.md
- references/webmcp-overview.md
- references/webmcp-proxy.md
- references/webmcp-react.md
- references/webmcp-security.md
- references/webmcp-timeline.md
- references/webmcp-user-journeys.md
- references/webmcp-vs-automation.md
- references/webmcp-vs-mcp.md
---

**Requires:** file-read, file-edit, terminal (local verification). Browser
devtools or an agent-capable browser recommended for end-to-end testing; not
required to write correct code.

**Full reference:** [webmcp-overview](references/webmcp-overview.md) ·
[webmcp-apis](references/webmcp-apis.md) ·
[webmcp-implementation-gotchas](references/webmcp-implementation-gotchas.md)

WebMCP is a Draft Community Group Report (W3C Web Machine Learning CG), not a
shipped standard. Chromium (`document.modelContext`) runs an origin trial from
Chrome 149; Firefox and Safari have no ship date. Treat every integration as
progressive enhancement, never a load-bearing feature.
[webmcp-timeline](references/webmcp-timeline.md)

## Preconditions

- Confirm the page already has the client-side logic you're exposing (a form
  handler, a search function, a cart mutation) — WebMCP wraps existing code,
  it does not replace writing the feature.
- Decide WebMCP vs a plain MCP server **per capability**, not once for the
  product: live tab + page/session state → WebMCP; unattended, CLI, or
  "anywhere anytime" → MCP; both can share one service layer.
  [webmcp-vs-mcp](references/webmcp-vs-mcp.md)

## Defaults (proceed without asking)

- Use the imperative API (`registerTool`) for app logic; use the declarative
  form attributes (`toolname`/`tooldescription`) only when a plain HTML form
  *is* the whole tool.
- Feature-detect `document.modelContext` and **poll briefly** rather than
  checking once — see Key rules below.
- **Always ask:** whether a mutating/high-stakes tool needs a
  `consequentialHint` and human confirmation flow, if the product's risk
  tolerance for that action isn't already established elsewhere in the repo.

## Key rules

### Registration (get this right first — most bugs live here)

- **Feature-detect with a poll, not a single check.** In real browsers,
  `document.modelContext` can attach *after* a synchronous `<script>` has
  already run. A one-shot `if (!document.modelContext) return;` silently
  registers nothing, with no error. Poll for a few seconds before giving up.
  [webmcp-implementation-gotchas](references/webmcp-implementation-gotchas.md)
- **`execute` wraps the same code path the UI button already uses** — don't
  fork logic. Pass the `signal` from `execute(input, { signal })` into any
  `fetch`/long work so cancellation actually stops it.
  [webmcp-apis](references/webmcp-apis.md)
- **Register for current page state; unregister with `AbortSignal`** when the
  UI can no longer perform the action. Static registration on load is fine
  for simple pages; complex multi-state UIs need dynamic lifecycle.
  [webmcp-best-practices](references/webmcp-best-practices.md)
- **Local dev over plain HTTP may not expose the API even with the flag on**
  — the experimental testing flag has been observed to require a secure
  (HTTPS) context, unlike most other browser APIs' `localhost` exception.
  Don't conclude WebMCP "isn't working" from an HTTP dev server; verify
  against HTTPS.
  [webmcp-implementation-gotchas](references/webmcp-implementation-gotchas.md)

### Tool design

- **One job per tool.** Overlapping tools compete for selection and confuse
  the model. Distinguish immediate execution from starting a flow
  (`create_event` vs `start_event_creation`).
- **Stay under budget.** Lighthouse warns above ~40 registered tools on a
  page; keep names ~30 chars, tool descriptions ~500 chars, parameter
  descriptions ~150 chars.
- **Design from outcomes, not your API surface.** Wrapping every endpoint
  produces a confused agent. Work backwards from the user goal, walk the
  turn-by-turn conversation, and only then decide which functions become
  tools. Classify each as **Answer** (low risk: search, read state),
  **Action** (medium: add to cart, save draft), or **Sensitive** (high: buy,
  publish, cancel — needs confirmation). Ship Answer tools first.
  [webmcp-user-journeys](references/webmcp-user-journeys.md)
- **The browser does not sync the UI for you.** WebMCP runs `execute` in the
  page, but nothing about the API repaints the DOM afterward — that's on your
  code. If `execute` mutates state without also updating what the human sees,
  the human and the agent silently drift out of sync. Update the same visible
  UI a human action would.
  [webmcp-best-practices](references/webmcp-best-practices.md)
- **Schemas minimize model work.** Accept raw input and normalize in your own
  code — no mental math or string transforms required of the model. Prefer
  self-explanatory enum values (`"express"`) over opaque IDs (`1`).
- **Validate strictly in code, loosely in schema.** Return actionable error
  strings from `execute` so the agent can self-correct and retry, rather than
  stalling on a rigid schema failure.

### Security (the trust boundary doesn't move)

- **If a capability would be unsafe as a public endpoint called by a hostile
  client, it is unsafe as a WebMCP tool.** A tool runs in the page with the
  user's real session and cookies — that's the value and the risk. Auth,
  ownership, quotas, and rate limits stay server-side, exactly as they would
  for any other client-callable endpoint.
  [webmcp-security](references/webmcp-security.md)
- **Set annotation hints honestly:** `readOnlyHint`, `untrustedContentHint`
  (output includes UGC or external data), `consequentialHint` (high-stakes or
  irreversible). Agents use these to decide when to ask for confirmation.
- **Tools default to same-origin only.** Cross-origin needs *both*
  `exposedTo: [origin]` on `registerTool` *and* `getTools({ fromOrigins:
  [...] })` on the caller, plus `allow="tools"` Permissions Policy on any
  cross-origin iframe. Only expose to origins you'd trust with that data or
  action — prefer same-origin unless there's a real partner/embed need.
  [webmcp-cross-origin](references/webmcp-cross-origin.md)

### Verifying it actually works (do not trust "no errors")

- **A 200 response is not proof the real script shipped.** A build step that
  forgets to copy the tool-registration file into the deployed output can
  fail completely silently on a host with SPA-style fallback routing — the
  fallback serves your app shell with HTTP 200 instead of erroring. Confirm
  the file is actually present in the *build output directory*, and navigate
  directly to the script's URL to confirm it's real JS, not your HTML shell.
  [webmcp-implementation-gotchas](references/webmcp-implementation-gotchas.md)
- **Check both cache layers when verifying a fix landed:** CDN edge cache can
  keep serving a stale (possibly wrong) response for the asset's full
  `Cache-Control: max-age`; the browser's own disk cache can do the same on
  a fresh tab. Use a cache-busting query to check the origin, and a hard
  reload (Ctrl/Cmd+Shift+R) to bypass the browser cache, before concluding a
  deploy did or didn't fix something.
- **End-to-end verification:** `const tools = await
  document.modelContext.getTools(); await
  document.modelContext.executeTool(tools.find(t => t.name === '…'), input)`
  — this is what a real browsing agent does; use it directly in devtools to
  confirm a tool actually works before trusting it in front of a model.

### If you show a "supports WebMCP" indicator in the UI

- **Gate visibility on actual tool registration succeeding, not just on
  `document.modelContext` existing.** Feature-detection alone can be true
  while your registration still silently failed (see Registration above).
- **Watch CSS specificity with the `hidden` attribute.** An element that both
  sets `display` in a class rule and toggles the `hidden` attribute needs an
  explicit `.your-badge[hidden] { display: none }` override — otherwise the
  author rule's equal specificity beats the browser's built-in
  `[hidden]{display:none}` by cascade order, and the element shows
  regardless of `hidden`.
  [webmcp-implementation-gotchas](references/webmcp-implementation-gotchas.md)

## Output format

Write code changes directly. After edits, state which tools were
registered/changed, which annotations/hints were set and why, and how you
verified registration actually succeeds (not just that the code compiles).
For question-answering: concise prose with inline code, citing the specific
reference page for anything non-obvious.

## Error handling

- If `document.modelContext.getTools()` stays empty with no thrown error
  after your change, do not assume it's a security/annotation problem before
  ruling out the registration-timing and deployment gotchas above — they
  produce exactly this symptom.
- If a build/deploy pipeline is involved, verify the registration script is
  present in the actual build output before debugging the JavaScript further.
- Never silently swallow a `registerTool` rejection — log it, since
  `NotAllowedError` (permissions policy) and schema errors both throw there.

## Validation

```bash
# No dedicated WebMCP linter; validate with the project's normal build/serve
# commands, then confirm end-to-end in a WebMCP-capable browser:
#   1. Build and serve the real (not dev-shortcut) output.
#   2. Hard-reload the target page.
#   3. In devtools: await document.modelContext.getTools()
#   4. Execute one tool via document.modelContext.executeTool(...) and
#      confirm the visible UI updates the same way the human control does.
```

## Evaluations (I/O examples)

**Input:** "Add a WebMCP tool so an agent can add an item to the cart"
**Expected:** Registers `add_to_cart` wrapping the existing add-to-cart
function, `annotations: { readOnlyHint: false }`, schema with product id/qty,
feature-detects with a poll (not a single check), and reports how it verified
`getTools()`/`executeTool()` end-to-end.

**Input:** "Our WebMCP tools worked yesterday, now `getTools()` returns
empty and there are no console errors"
**Expected:** Checks, in order: (1) does the deployed script actually exist
at that URL as real JS, not an SPA-fallback HTML page; (2) is this a stale
CDN/browser cache serving the old broken state; (3) is registration
happening before `document.modelContext` attaches. Does not guess at
security/permissions causes first, since those would normally throw.

**Input:** "Should this be a WebMCP tool or an MCP server?"
**Expected:** Asks whether the capability needs a live tab and page/session
state (→ WebMCP) or must be reachable unattended/from non-browser clients
(→ MCP), per the decision order in
[webmcp-vs-mcp](references/webmcp-vs-mcp.md), rather than picking one
architecture for the whole product.

## Skill handoffs

- Plain backend MCP server work with no browser page involved → this skill
  does not apply; that's ordinary MCP server implementation.
- React-specific registration lifecycle →
  [webmcp-react](references/webmcp-react.md) section of this skill's own
  references; no separate skill needed.
- General frontend conventions unrelated to WebMCP (component structure,
  state management) → [paniolo-react-best-practices/SKILL.md](../paniolo-react-best-practices/SKILL.md).

## Do Not

- Do not conclude a browser "doesn't support WebMCP" from a plain HTTP
  `localhost` dev server — check against HTTPS first.
- Do not trust a 200 status on the tool-registration script's URL as proof it
  deployed correctly — navigate to it directly and confirm it's real JS.
- Do not check `document.modelContext` exactly once and give up if it's
  undefined — poll briefly; the browser may attach it a moment later.
- Do not skip server-side auth/ownership/quota checks inside `execute`
  because "it's just a browser tool" — it rides the user's real session.
- Do not register write/sensitive tools without `consequentialHint`, and do
  not assume `readOnlyHint` when the tool can mutate state.
- Do not expose tools cross-origin (`exposedTo`) to an origin you would not
  otherwise trust with that user's data or actions.
- Do not treat WebMCP as a stable, universally supported feature — it is an
  origin-trial-stage draft on Chromium only as of this writing.

## References

- [webmcp-overview](references/webmcp-overview.md) — what WebMCP is and isn't
- [webmcp-vs-mcp](references/webmcp-vs-mcp.md) — decision order for WebMCP vs MCP
- [webmcp-vs-automation](references/webmcp-vs-automation.md) — WebMCP vs DOM automation vs MCP Apps
- [webmcp-apis](references/webmcp-apis.md) — imperative/declarative API mechanics
- [webmcp-best-practices](references/webmcp-best-practices.md) — tool strategy, naming, schemas
- [webmcp-security](references/webmcp-security.md) — hints, origin gating, agent-side defenses
- [webmcp-cross-origin](references/webmcp-cross-origin.md) — `exposedTo`/`fromOrigins` gates
- [webmcp-react](references/webmcp-react.md) — `usewebmcp` and component lifecycle
- [webmcp-proxy](references/webmcp-proxy.md) — bridging a remote MCP server onto `document.modelContext`
- [webmcp-user-journeys](references/webmcp-user-journeys.md) — designing tools from CUJs, not endpoints
- [webmcp-evals](references/webmcp-evals.md) — testing agent tool selection, not just JS correctness
- [webmcp-observability](references/webmcp-observability.md) — production signals without PII
- [webmcp-timeline](references/webmcp-timeline.md) — browser/flag rollout status
- [webmcp-implementation-gotchas](references/webmcp-implementation-gotchas.md) — silent-failure bugs seen in production
- [paniolo-react-best-practices/SKILL.md](../paniolo-react-best-practices/SKILL.md)
