---
source-slug: mcp-skills-extension
source-hash: 48dfc728e4e5bbbfb831838c92109d991d11c108857df591aaf13c5cb3516b22
bundled: 2026-09-30
title: Skills over MCP (SEP-2640)
type: concept
tags:
- mcp
- skills
- agents
- specification
updated: 2026-09-30
---

# Skills over MCP (SEP-2640)

**SEP-2640 (Skills Extension), Status: Final, Extensions Track** — created
2026-04-23 by the Skills Over MCP Working Group. Extension identifier
`io.modelcontextprotocol/skills`; negotiated per SEP-2133 under
`capabilities.extensions`. It defines a **transport binding only**: how a
server ships Agent Skills over MCP. The skill format itself — directory
layout, `SKILL.md`, YAML frontmatter, naming rules, progressive disclosure —
is delegated entirely to the Agent Skills specification (snapshot under
`raw/harness-eng/agentskills-io/`) and follows its future revisions.

The problem it solves: a server and the skill teaching an agent to use it were
versioned, discovered, and installed separately; server `instructions` are
practically size-bounded (an 875-line `SKILL.md` does not fit); and
independent implementations had each invented divergent `skill://` URI
structures.

## Mechanism

Skills ride the existing **Resources** primitive — no new resource type. Every
file in a skill directory is an individually addressable MCP resource.

Three protocol methods:

| Method | Required | Purpose |
| --- | --- | --- |
| `skills/list` | yes | Paginated enumeration of skill entries |
| `skills/get` | yes | Entry for one skill by URI, incl. skills absent from the listing |
| `resources/directory/read` | opt-in (`directoryRead` setting) | Direct children of a directory resource — scoped navigation of `templates/`, `scripts/` |

A **skill entry** is a complete manifest, not a summary: verbatim `frontmatter`
(every field the author wrote, so a host builds its registry from the listing
alone, no `SKILL.md` fetch), the `uri`, and `resources` — either a complete
`{uri, digest, size}` array covering every file, or the string `"dynamic"`
for generated content. Digests are `sha256:{hex}` over raw bytes.

Fixed limits: **512 resources and 16 MiB per skill**, both checkable from the
entry alone before fetching a byte.

## URIs and identity

`skill://<skill-path>/<file-path>` — `SKILL.md` is always explicit in the URI
(`skill://acme/billing/refunds/SKILL.md`). `<skill-path>` may nest; its final
segment MUST equal the frontmatter `name`, so the name is recoverable from the
URI without a fetch. The scheme is conventional, not privileged: a server MAY
serve skills under `github://` or any scheme, and a host MUST NOT conclude a
resource is a skill from its URI shape — skill-ness comes from a `skills/list`
entry or a `skills/get` confirmation.

**Skill identity is the pair (host-assigned server identity, `uri`).** Two
servers can both serve `skill://refunds/SKILL.md`; those are unrelated skills.
Registries, persisted approvals, caches, materialized paths, and every
model-facing reference MUST key on both halves — never the URI alone.

Enumeration may be **empty or partial** by design (generated catalogs,
gateways); hosts MUST NOT read it as proof of absence, which is what
`skills/get` is for. `skill://index.json` from earlier drafts is gone —
replaced by `skills/list` for pagination, uniform caching attributes
(SEP-2549 `ttlMs`/`cacheScope`), and declaration-based discoverability.

## Reading is not loading

`resources/read` of a `SKILL.md` is transport — it returns bytes and does not
activate the skill. Activation goes through the host's skill-loading path,
which verifies content against the entry, applies user approval, and opens the
window in which the host is *acting on* the skill. While acting, reads resolve
only to URIs in the held entry's `resources`; an unlisted file is a
verification failure equivalent to a digest mismatch. Hosts fetch on demand —
never the catalog upfront — and `skills/get` refreshes one entry after a
mismatch without re-enumerating.

## Security model

The SEP's Security Implications are unusually prescriptive — skill content is
server-authored instructional text, a prompt-injection surface, and can place
bytes on the host filesystem that the model is then directed to execute.

- Skill content is **untrusted input**; connection does not confer authority.
- **Origin MUST be visible to the model** — an MCP skill MUST NOT be presented
  as indistinguishable from a filesystem skill.
- **No implicit local execution** — explicit per-skill approval gates both
  declarative fields (hooks, frontmatter scripts) and body instructions that
  direct host code-execution tools.
- **Origin-scoped reads** — a model-callable resource reader is a cross-server
  confused deputy: a skill from server A MUST NOT cause a `resources/read`
  against server B. Servers are identified by host-assigned labels, never
  self-reported `serverInfo.name`.
- **Name collisions impersonate** — resolve names per-origin; no silent
  shadowing of same-named skills; surface collisions to the user.
- **`allowed-tools` is ignored** for MCP-origin skills unless the user
  approved that grant — a remote server setting it is requesting host access.
- **Content-bound approval** — persisted approval binds to the `resources`
  set observed at approval time; any changed `uri`/`digest` revokes it.
  `"dynamic"` skills cannot be content-bound; hosts MAY refuse them.
- **Digests are not a security boundary** — unsigned, same-origin; a rewriting
  intermediary stays consistent. They prove listing/content agreement, not
  trustworthiness.
- **Cache isolation** — on-disk caches live outside every filesystem-skill
  discovery path, stay MCP-origin forever (durable origin, surviving restart
  and disconnect), and must be host-private or re-hashed per access.
- **Nested skills** — a `SKILL.md` in a subdirectory is its own skill;
  approving the enclosing skill does not approve it.

The working group's archived threat model (snapshotted alongside) grounds each
requirement in a runnable adversarial corpus (`dangerous-skills-mcp`).

## Deferred and adoption notes

- **Archive distribution (tar/zip) was cut in review**: safe unpacking is a
  disproportionate attack surface (decompression bombs, path traversal,
  setuid bits, normalization collisions) and a second encoding breaks the
  flat compatibility floor — any conforming host reads any conforming skill.
- **Reference implementations**: SDK PRs for Python, C#, and Go; conformance
  scenarios (`sep-2640.yaml`); prototype hosts (gemini-cli, fast-agent,
  codex, and an internal Claude Code prototype); a GitHub MCP Server
  prototype. FastMCP's `SkillsProvider` diverges on URI structure and
  discovery — migration is a stated working-group priority.
- List-caching attributes apply only on protocol 2026-07-28+.

## Token-efficiency and Paniolo relevance

This is progressive disclosure made protocol-native — the model sees
`name`+`description` from a paginated listing, fetches `SKILL.md` on demand,
and pulls supporting files only when referenced (see [mcp-token-efficiency](./mcp-token-efficiency.md)).
It also answers the "ship instructions with tools" gap for any server whose
workflows outgrow the `instructions` field.

For `paniolo mcp` specifically: the server currently exposes tools only — no
`resources` capability, no extension declarations — so serving installed
skills over MCP is a new lane, not an extension of the existing surface. It
would mean declaring `resources` + `io.modelcontextprotocol/skills`,
implementing `resources/read`, `skills/list`, `skills/get` (and optionally
`resources/directory/read`), and meeting the host-side contract above on the
client side. The identity rule has a direct Paniolo consequence: local
installed skills and MCP-served skills must never silently shadow each other.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
