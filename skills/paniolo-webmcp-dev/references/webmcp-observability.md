---
source-slug: webmcp-observability
source-hash: 3f60e6c1fea0223739f966b8ef1f58777645c1205d743d51eb81157cfe940c07
bundled: 2026-09-25
title: WebMCP Observability
type: concept
tags:
- webmcp
- observability
- telemetry
- privacy
updated: 2026-09-25
---

# WebMCP Observability

Once tools leave local flags, you need signals that separate **model mistakes**,
**tool bugs**, and **downstream API** failures — without turning telemetry into
a second copy of user PII.

## What to measure

- Invocations per tool; latency; success / failure / cancel rates
- Validation failures vs application errors
- Journey drop-offs (which tool was last before abandon)
- Mid-chain failures that still "succeed" overall (e.g. skipped discount)

Production logs feed [webmcp-evals](./webmcp-evals.md) datasets and tool description fixes
([webmcp-user-journeys](./webmcp-user-journeys.md)).

## Privacy default

**Do not record raw tool arguments and results by default.** Song lyrics,
addresses, and account data will show up in WebMCP traffic. Prefer aggregates,
error codes, and redacted shapes. If you need sample payloads for debugging,
gate them, sample them, and expire them.

## Intent visibility (site-owner upside)

When agents call tools, you see structured demand (query params, filters,
goals). When they scrape the DOM, you don't. That first-party intent signal is
a product reason to prefer WebMCP over silent scraping — still under privacy
constraints above.

## Implementation notes

- Wrap `execute` in your existing tracer/spans; no standard WebMCP OTel
  conventions yet — keep attribute names boring.
- Distinguish cancel (`AbortSignal`) from failure in metrics.
- Tie dashboards to tool layers (Answer / Action / Sensitive) so risk is
  visible.
