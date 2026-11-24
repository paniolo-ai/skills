---
name: paniolo-wiki-cloud
description: |
  Use Paniolo Wiki Cloud — the zero-install web editor (paniolo.ai/dashboard/wiki), Browser WebMCP (document.modelContext), and Remote MCP server (api.paniolo.ai/v1/wiki/mcp) over your own GitHub wiki repositories, with browser-local QMD search. Use when connecting GitHub repos, minting a Remote MCP Bearer token, calling Remote MCP or WebMCP wiki tools, ingesting HTTPS URLs into raw/, or configuring browser-local embeddings and reranking. Do NOT use for local CLI wiki validation or editorial schema rules (load paniolo-wiki instead).
license: MIT
metadata:
  version: 0.1.0
tags:
- wiki
- cloud
- mcp
- webmcp
- qmd
user-invocable: true
references: []
---

**Requires:** a Paniolo account (`https://paniolo.ai/login`), the `Paniolo Wiki` GitHub App installed on one or more wiki repositories, and either a web browser or an MCP-capable client.

# Paniolo Wiki Cloud

**Paniolo Wiki Cloud** is the zero-install hosted surface for GitHub-backed LLM wikis: a web editor at `https://paniolo.ai/dashboard/wiki`, **Browser WebMCP** (`document.modelContext`), and a **Remote MCP** endpoint at `https://api.paniolo.ai/v1/wiki/mcp`. Your Markdown lives in your own GitHub repositories (no cloud lock-in), and interactive search, embeddings, reranking, graph layout, and health scans run locally in your browser.

For wiki authoring discipline (page kinds, `raw/` provenance, `wiki/log.md` verbs, and local CLI validation), load `paniolo-wiki`.

## 1. Connect GitHub Repositories

1. Sign in at `https://paniolo.ai/login` (Google or GitHub OAuth) and open `https://paniolo.ai/dashboard/wiki`.
2. Click **Connect GitHub Repos** to install the **Paniolo Wiki** GitHub App on the repositories you want to manage.
3. GitHub redirects back with `?installation_id=...`, which claims per-account repository grants. Every read, write, and Remote MCP call is isolated to the signed-in account's grants.

## 2. Remote MCP Server (`https://api.paniolo.ai/v1/wiki/mcp`)

Use Remote MCP to let headless AI agents (Claude Desktop, Cursor, Codex, Antigravity) read, search, capture sources, and propose Pull Requests against your granted GitHub wikis without a local checkout.

### Authenticate with a Scoped Bearer Token

1. In `https://paniolo.ai/dashboard/wiki`, open **Settings** → **Remote MCP Server & Browser WebMCP**.
2. Click **Generate Scoped MCP Bearer Token (30d)** (calls `POST /v1/wiki/mcp/token`, scopes: `wiki:read`, `wiki:capture`, `wiki:propose`).
3. Paste the generated config into your MCP client:

```json
{
  "mcpServers": {
    "paniolo-wiki": {
      "url": "https://api.paniolo.ai/v1/wiki/mcp",
      "headers": {
        "Authorization": "Bearer <your-scoped-token>"
      }
    }
  }
}
```

RFC 9728 protected-resource metadata is published at `GET https://api.paniolo.ai/v1/wiki/mcp/.well-known/oauth-protected-resource`.

### Remote MCP Tool Catalog (JSON-RPC 2.0 · `2025-03-26`)

| Tool | Scope | Behavior |
| --- | --- | --- |
| `paniolo_wiki_list_projects` | `wiki:read` | Lists GitHub wiki repositories granted to the authenticated account. |
| `paniolo_wiki_list_pages` | `wiki:read` | Lists Markdown files in a granted `(installation_id, owner, repo, branch, root)`. |
| `paniolo_wiki_read_page` | `wiki:read` | Reads a commit-pinned Markdown file (`path`, `sha`, `content`). |
| `paniolo_wiki_search_keyword` | `wiki:read` | Searches Markdown paths/slugs in a granted repository. |
| `paniolo_wiki_capture_source` | `wiki:capture` | SSRF-guarded live HTTPS fetch returning a provenance-stamped `raw/<domain>/<slug>.md` preview. |
| `paniolo_wiki_propose_change` | `wiki:propose` | Opens a **GitHub Pull Request** with proposed file operations. Never pushes directly to the default branch. |
| `paniolo_wiki_job_status` | `wiki:read` | Inspects an account-bound wiki job status. |

**Safety invariant:** Remote MCP agents can never self-approve or push directly to the default branch; `paniolo_wiki_propose_change` always creates a topic branch and Pull Request for human merge.

## 3. Browser WebMCP (`document.modelContext`)

When you open `https://paniolo.ai/dashboard/wiki` in a WebMCP-enabled browser (e.g., Chrome with `document.modelContext` or `paniolo browser launch`), the page automatically registers **9 live browser tools** and shows a `WebMCP: 9 tools` badge in the status bar:

- **Read & Navigate:** `paniolo_wiki_current_context`, `paniolo_wiki_list_pages`, `paniolo_wiki_read_page`, `paniolo_wiki_find_pages`, `paniolo_wiki_scan_corpus`, `paniolo_wiki_related_pages`, `paniolo_wiki_job_status`.
- **Stage & Propose:** `paniolo_wiki_stage_draft` (updates the active editor buffer) and `paniolo_wiki_propose_change` (queues a diff into the visible **Review Inbox** — requires explicit human approval before committing or opening a PR).

## 4. Web Editor & Browser-Local QMD

All editing, search, and health checks run in the browser against your synced GitHub pages (cached per-account in IndexedDB):

- **Editing & Publishing:** Obsidian-style CodeMirror 6 Live Preview, `[[wikilink]]` autocomplete across all project wikis, **Direct to default branch** vs. **Open Pull Request** toggle, structured **+ New Page** modal (canonical kind prefixes + slug collision check), and atomic **Page Ops** (Rename with automatic `[[wikilink]]` rewriting, Archive to `raw/wiki-archive/`, Delete with backlink guard).
- **Navigation & Health:** **Command Palette** (`Ctrl/Cmd+P`), **Find Pages** (`Ctrl/Cmd+K`), **Scan** tab, **Graph** tab, **Page Stats** modal, **Backlinks**, **Related Pages & Suggested Links** (whole page or editor selection, with 1-click `+ Link`), and **Ingest URL** (SSRF-guarded `raw/` snapshot + `sources:` attachment).
- **Browser-Local QMD Presets:**
  - **Lexical + Fuzzy (0 MB, default):** Instant offline BM25 + typo-tolerant fuzzy search.
  - **Lite (~56 MB · WebGPU/WASM):** Local ONNX `bge-small-en-v1.5` embeddings + `ms-marco-MiniLM-L-6-v2` cross-encoder reranking.
  - **QMD Parity (~310 MB · Desktop WebGPU):** Local ONNX `embeddinggemma-300m` embeddings + `bge-reranker-base` cross-encoder reranking (auto-falls back to Lite/WASM on mobile or non-WebGPU devices).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
