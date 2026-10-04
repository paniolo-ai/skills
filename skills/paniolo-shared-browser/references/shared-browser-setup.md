---
source-slug: shared-browser-setup
source-hash: 256ba33cfc9f416211e062448fc157f7b6bcf4457639decaf1eb31145d6d0ce4
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

Requests such as "open this site in a browser we both control" should select
the shared-browser skill automatically. The agent handles setup and opening
the supplied site; the human does not need to supply launch flags or name the
MCP server. Reuse the dedicated persistent profile so previous sign-in survives.

## Connect The Agent Before Opening Chrome

Configure Chrome DevTools MCP in the agent host, then reload that host's MCP
connection. For authenticated browsing, launch normal Chrome with a dedicated
profile, then attach the MCP server to it. Some accounts reject sign-in when
the MCP server launches Chrome in automation mode. WebMCP is optional, and
the browser can visit local or public websites. Node.js and npm must be
available to the agent host.

For hosts that load project `.mcp.json`, merge this entry with existing servers
on macOS or Linux:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@1.10.1", "--browserUrl", "http://127.0.0.1:9222", "--no-usage-statistics"]
    }
  }
}
```

On native Windows, use `cmd.exe` to run the npm command shim:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "cmd.exe",
      "args": ["/c", "npx", "-y", "chrome-devtools-mcp@1.10.1", "--browserUrl", "http://127.0.0.1:9222", "--no-usage-statistics"]
    }
  }
}
```

Do not add `--headless` or `--isolated`: the human needs the window and a
profile that retains sign-in between sessions. Launch the dedicated browser
before calling page tools; the port must match `--browserUrl`. Use the same
MCP connection throughout the task. Port 9222 is an example, not a website
restriction or a requirement of the protocol.

## Register With The Actual Agent Host

A repository `.mcp.json` is not a universal agent configuration format.
Preserve existing servers and register the equivalent command where the host
loads MCP servers:

Installing the skill supplies instructions and references. It does not install
the browser MCP server configuration. Add or merge the configuration for every
AI host the customer harness supports; do not replace unrelated server entries.
A harness supporting Codex and Claude Code needs both configuration files below,
unless the equivalent server is already configured at user scope.

| Agent Host | Configuration |
| --- | --- |
| Claude Code | Project `.mcp.json`; enable the project server in the host |
| Codex | Project `.codex/config.toml` in a trusted project, or user `~/.codex/config.toml` |
| Cursor | Project `.cursor/mcp.json`, using the `mcpServers` JSON examples above |
| Copilot In VS Code | Project `.vscode/mcp.json`, using `servers` rather than `mcpServers` |
| Gemini CLI | Project `.gemini/settings.json`, merging a `mcpServers` object |
| Other MCP Hosts | Use that host's MCP settings and its supported command format |

Cursor and Gemini CLI use the same `chrome-devtools` object shown in the JSON
examples above, inside their existing `mcpServers` object. VS Code needs this
native Windows example instead:

```json
{
  "servers": {
    "chrome-devtools": {
      "type": "stdio",
      "command": "cmd.exe",
      "args": ["/c", "npx", "-y", "chrome-devtools-mcp@1.10.1", "--browserUrl", "http://127.0.0.1:9222", "--no-usage-statistics"]
    }
  }
}
```

On macOS or Linux, use `npx` as the command and remove `"/c", "npx"` from
that argument list. Copilot CLI is a separate host from Copilot in VS Code;
use its own MCP configuration rather than assuming it reads `.vscode/mcp.json`.
For Antigravity, Devin, or another host, check its current MCP settings and local
desktop connection support. Do not invent a configuration path or assume a
remote agent can reach the human's loopback Chrome port.

Vendor references: [Cursor MCP](https://prod.cursor.com/help/customization/mcp),
[VS Code MCP](https://code.visualstudio.com/docs/agent-customization/mcp-servers),
and [Gemini CLI MCP](https://geminicli.com/docs/tools/mcp-server/).

Codex on native Windows:

```toml
[mcp_servers.chrome-devtools]
command = "cmd.exe"
args = ["/c", "npx", "-y", "chrome-devtools-mcp@1.10.1", "--browserUrl", "http://127.0.0.1:9222", "--no-usage-statistics"]
startup_timeout_sec = 60
```

On macOS or Linux, use `command = "npx"` and omit `"/c", "npx"` from the
arguments. Run the server in the desktop host's environment. A WSL server
launches a Linux browser; it does not automatically share a Windows Chrome
window. If the host runs remotely, first establish its supported local MCP
connection rather than exposing a debugging port on the network.

Reload the host after changing configuration. Then verify `list_pages`,
`navigate_page`, and `take_snapshot` are present in the agent's actual tool
catalog. Launch the dedicated browser, call `list_pages`, navigate to the requested URL,
and take a snapshot. A config file, a successful package invocation, or a
responding `/json/version` endpoint alone does not verify this connection.

Windows verification on 2026-10-03: server version 1.10.1 completed an MCP
handshake, listed navigation and editing tools, and `list_pages` launched a
visible Chrome window. A subsequent interactive test loaded BardoShare over
HTTPS, but Google rejected sign-in in that server-launched browser. A normal
Chrome launch with a dedicated profile then exposed a working loopback debug
endpoint. The human then successfully signed in, and an attached MCP client
read the signed-in song list and opened Amazing Grace's edit page. The
agent host's existing server still used its previous launch arguments until
reload; writing the configuration did not retarget that running process.
macOS, Linux, and WSL were not exercised in these checks.

## Launch The Dedicated Shared Browser

The configuration above attaches instead of launching Chrome. The browser
must be running and its endpoint must respond before page tools can work.
Start normal Chrome directly; let the human sign in inside that shared
profile. Do not transfer cookies from another profile or disguise automation.

### Manual Launch

Start Chrome with three things:

1. `--remote-debugging-port=9222` serves CDP on loopback.
2. `--user-data-dir=<dedicated dir>` gives the browser its own profile.
3. When WebMCP is requested, add `--enable-features=WebMCP,WebMCPTesting`.
   This combination passed registration, discovery, and execution on the
   shared Windows Chrome instance. Ordinary website navigation does not need it.

Example on Windows (PowerShell):

    Start-Process "C:\Program Files\Google\Chrome\Application\chrome.exe" `
      -ArgumentList "--remote-debugging-port=9222", `
        "--remote-debugging-address=127.0.0.1", `
        "--user-data-dir=`"$env:USERPROFILE\.shared-browser`"", `
        "--no-first-run", "--enable-features=WebMCP,WebMCPTesting", `
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
profile, and the new flags are dropped. If the dedicated shared browser's
endpoint already works, reuse it and open the requested URL through MCP.
Restart that profile only when its required launch flags were missing; do not
close a working shared browser just to open another website.

When the agent's page list does not match the human's window, check the active
server's connection mode. A running MCP server keeps its launch arguments
until the host reloads it. Retarget and verify the actual server before
continuing; do not create another browser and assume it shares the session.

## Check The Attached Browser

    curl http://127.0.0.1:9222/json/version

A JSON body with a `webSocketDebuggerUrl` means CDP is up. Still call the
agent's MCP `list_pages` and take a snapshot to verify its connection.

## Verify WebMCP

When WebMCP is requested, use HTTPS and verify it in the actual shared tab
through MCP `evaluate_script`. Poll briefly for `document.modelContext`, then
check that `registerTool`, `getTools`, and `executeTool` are functions. A
working debug port does not prove these browser APIs are enabled.

If the API stays absent, gracefully close only the dedicated shared browser
after checking for unsaved work. Relaunch the same profile with the WebMCP
flags above and reopen the requested URL. Starting Chrome again while that
profile is still open does not apply new flags. Preserve the profile and sign-in.

Wait separately for the app's actual editor or other relevant controls to be
ready, then call `getTools()`. API availability and site registration are
different checks. Do not conclude that the site exposes no tools from an empty
list during loading. Poll registration for a bounded interval and report a
timeout rather than claiming the site has no integration.

Return tool names or selected metadata from `evaluate_script`, rather than
whole registered tool objects: those can include circular references to Window.
Execute a documented read-only site tool using its registered object and
`JSON.stringify` arguments; see [shared-browser-page-tools](./shared-browser-page-tools.md). Verify the result
matches the visible page. Do not mutate customer data merely to test connectivity.

Observed on 2026-10-03: after restarting shared Chrome with both feature flags,
the HTTPS BardoShare song editor exposed 21 site tools once loading completed.
`read_bardoshare_song_editor` executed successfully for Amazing Grace and
returned saved status. No song content was changed. An earlier empty discovery
result occurred before the editor was ready.

## Teardown

Close the window, or stop only the processes whose command line contains the
dedicated profile path. Do not kill every `chrome.exe`: that closes the
human's other windows and can lose unsaved work.

## Server-Managed Launch Without Sign-In

For tasks that do not need authentication, omit `--browserUrl` and its value
from the server arguments. `list_pages` then launches a visible Chrome window
with a persistent dedicated profile and no fixed port requirement. This mode
passed the startup test, but it is insufficient evidence for sign-in support.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
