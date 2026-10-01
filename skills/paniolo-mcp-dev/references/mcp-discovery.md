---
source-slug: mcp-discovery
source-hash: c35cd8ce015d4ca59ef2acaba61835dcb98e277577ffd31f969d08fe36542d2f
bundled: 2026-09-30
title: MCP Discovery — registries, capabilities, and catalogs
type: concept
tags:
- mcp
- discovery
- registry
- agents
updated: 2026-09-30
---

# MCP Discovery — registries, capabilities, and catalogs

"How does an agent find out what a server can do?" has three answers at three
different layers, and they fail differently. Design for all of them.

## Layer 1 — install-time: the registry

The official **MCP Registry** is the app-store layer: a server publishes a
`server.json` manifest and clients/hosts list it for discovery. Status at
snapshot: **preview with an API freeze at v0.1** — stable for integrators, GA
pending. Publishing goes through the `mcp-publisher` CLI; namespaces are
authenticated (GitHub OAuth for `io.github.*` namespaces).

This is *user* discovery — it answers "which server do I install," not "what
can it do." A listing carries no capability truth; the server still has to
declare them at connect time. Keep the manifest honest (names, package refs,
remote endpoints) because it's the only signal a user sees before connecting.

## Layer 2 — connect-time: capability negotiation

`initialize` is where the server declares what it actually supports —
`tools`, `resources`, `prompts`, `sampling`, `elicitation`, and extension
identifiers under `capabilities.extensions` (SEP-2133; e.g.
`io.modelcontextprotocol/skills` for SEP-2640). The discipline:

- **Declare only what you implement.** A capability is a promise the client
  will act on — declaring `resources` and erroring on `resources/read` is
  worse than not declaring it.
- **Check before calling, never probe.** A client must not call a method the
  server didn't declare; a server must not assume a client capability the
  `initialize` result didn't carry. Extension methods are doubly gated —
  declare the extension, then the feature flag inside it (e.g.
  `directoryRead` for `resources/directory/read`).
- **Negotiated protocol version matters.** Dated versions through 2025-11-25
  use the stateful lifecycle; the 2026-07-28 draft moves to a **stateless**
  model where every request carries its own `_meta` and no connection or
  process identity implies session continuity. Don't build assumptions into
  connection lifetime.
- **Host support is uneven.** The clients feature matrix shows resources,
  sampling, elicitation, and extension support varying wildly by host — a
  mechanism that depends on an undeclared capability is dead on that host;
  degrade, don't fail.

## Layer 3 — in-session: the catalogs

Once connected, the model discovers through list methods, all paginated by
`cursor`/`nextCursor`:

| Method | Yields | Watch for |
| --- | --- | --- |
| `tools/list` | name + description + inputSchema + annotations | standing schema cost — every entry sits in context every turn |
| `resources/list` + `resources/templates/list` | addressable data + parameterized URIs | can be unbounded; list templates for generative URIs |
| `prompts/list` | canned workflow templates | user-facing in most hosts (slash commands) |
| `skills/list` | skill manifests (SEP-2640) | MAY be empty/partial — not proof of absence |

Two in-session dynamics matter:

- **`*/list_changed` notifications** let a server signal catalog mutation —
  declare `listChanged` on the capability and emit it, so hosts refresh
  instead of caching a stale tool list.
- **Descriptions are the discovery index.** The model picks tools by
  name+description at call time — a tool that can't be found by its
  description is a tool that doesn't exist. This is the same craft as
  registry listing text, one layer down.

## Progressive disclosure at scale

Flat catalogs stop working at hundreds of entries — the schema cost alone is
prohibitive (see [mcp-token-efficiency](./mcp-token-efficiency.md)). The escape patterns:

- **Search meta-tool**: expose `search_tools`/`find_capability` that returns
  names+descriptions; the full schema loads only on the call.
- **Code-mode file trees**: tools presented as a browsable filesystem —
  discovery becomes `ls`/`grep` over a directory, not a context-resident
  list.
- **Resources as documentation**: usage manuals and per-domain detail ship as
  resources (or SEP-2640 skills) fetched on demand, not in tool descriptions.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
