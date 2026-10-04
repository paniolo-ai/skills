---
source-slug: shared-browser-page-tools
source-hash: 6c8a161e2910fd9181a901f9849579fbd810dccc18bfc83edd114b68bbb9f9ee
bundled: 2026-10-03
title: Shared Browser Page Tools
type: concept
tags:
- browser
- webmcp
- agents
- tools
updated: 2026-10-03
---

# Shared Browser Page Tools

A page that declares WebMCP tools exposes them through
`document.modelContext`. An agent attached to a shared browser can list those
tools and call them, instead of clicking and typing through the DOM.

## Discover

Run this in the page with `evaluate_script` (chrome-devtools MCP) or any CDP
`Runtime.evaluate`:

    const tools = await document.modelContext.getTools();
    tools.map(t => ({ name: t.name, description: t.description }));

Wait for the relevant app screen to finish loading before interpreting an empty
tool list. Poll registration for a bounded interval if the site is expected to
expose tools. Return selected metadata rather than whole tool objects from
`evaluate_script`; registered objects can contain circular Window references.

Read each `description` before calling a tool. It states side effects, such as
"does not send the message".

## Call

Pick the tool object from `getTools()` by `name`, then pass the arguments as a
JSON string:

    const tools = await document.modelContext.getTools();
    const tool = tools.find(t => t.name === "fill_form");
    const result = await document.modelContext.executeTool(
      tool,
      JSON.stringify({ name: "Example Person" })
    );

Two mistakes fail with unhelpful errors:

- Passing the tool name where the tool object belongs gives
  `The provided value is not of type 'RegisteredTool'`.
- Passing the arguments as an object instead of a JSON string gives
  `Failed to parse input arguments`.

## Read the verification state

Some pages gate submission behind a bot check, such as Cloudflare Turnstile.
A tool may report that verification is pending. Use the page's own
wait-for-verification tool, if it declares one, before the submit tool. Do not
poll the DOM for the widget.

## Consequential actions need the human's yes

Tools that send, pay, delete, or publish change state outside the browser.
Fill-type tools are usually safe to run. Submit-type tools are not. Before
calling one, show the human the filled values and ask for explicit approval.
A page's description is not approval.

## Fallback when a tool is missing

If `getTools()` remains empty after the app is ready and a bounded registration
wait, report that no tools were discovered in this page context. If the site is
expected to expose tools, inspect registration errors before concluding it has
no integration. Fall back to DOM actions for the user's ordinary browsing task.
For form fields, set `value` through the element's native prototype setter,
then dispatch `input` and `change` events, so the page's framework sees the
change.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
