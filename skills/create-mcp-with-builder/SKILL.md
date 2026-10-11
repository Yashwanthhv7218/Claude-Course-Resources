---
name: create-mcp-with-builder
description: Guides a user through building a fully functional MCP server using the official mcp-server-dev builder plugin from the Claude plugins registry. This skill should be used whenever someone wants to create or scaffold an MCP server using Claude's built-in MCP builder tool. Trigger when users say "build an MCP server", "create an MCP using the builder", "use mcpbuilder", "use mcp-server-dev", "scaffold an MCP server for me", or "help me make an MCP integration with the builder tool".
---

# Create an MCP Server with the MCP Builder

You are guiding the user through building a fully functional MCP server using the official **`mcp-server-dev`** plugin — three composable skills that cover the full build path: discovery → scaffolding → packaging → testing.

---

## Step 1 — Check whether the builder is installed

Before anything else, run:

```bash
cat ~/.claude/plugins/installed_plugins.json
```

Look for a key starting with `"mcp-server-dev"` in the `plugins` object.

**If found** → skip to Step 3.  
**If not found** → proceed to Step 2.

---

## Step 2 — Install the plugin

Tell the user:

> The `mcp-server-dev` builder plugin is not installed. Run this command in your terminal (or prefix with `!` to run it here):
>
> ```
> claude plugin install mcp-server-dev@claude-plugins-official
> ```
>
> This installs three composable skills from the official Claude plugins registry:
>
> | Skill | Purpose |
> |---|---|
> | `build-mcp-server` | Entry point — interrogates the use case, picks deployment model, scaffolds the server inline or routes to a specialized skill |
> | `build-mcp-app` | Adds interactive UI widgets (forms, pickers, confirm dialogs, charts) rendered inline in chat |
> | `build-mcpb` | Packages a local stdio server with its runtime — users install one `.mcpb` file, no Node or Python required |
>
> Once installation completes, type **"continue"** and I'll pick up from here.

After the user confirms, re-check `installed_plugins.json`. If `mcp-server-dev` still doesn't appear, suggest running `claude plugin update` to refresh the marketplace index, then retrying.

---

## Step 3 — Start the builder

Invoke the entry-point skill:

```
/mcp-server-dev:build-mcp-server
```

`build-mcp-server` runs a discovery conversation and then **either scaffolds inline or routes to a specialized skill** depending on the use case. Here is what it does and decides:

---

### Discovery questions (Phase 1 of the builder)

The builder asks four questions — batch them in one message:

1. **What does it connect to?**
   - Cloud API / SaaS → Remote HTTP (default)
   - Local filesystem, desktop app, OS-level APIs → MCPB
   - Nothing external / pure logic → either, defaults to remote

2. **Who will use it?**
   - Just you / your team on your machines → Local stdio is acceptable
   - Anyone who installs it → Remote HTTP or MCPB
   - Claude Desktop users who want UI widgets → MCP app

3. **How many distinct actions does it expose?**
   - Fewer than ~15 → one tool per action
   - Dozens to hundreds → search + execute pattern (`search_actions` / `execute_action`)

4. **Does any tool need mid-call user input or rich display?**
   - Simple confirm / pick from short list → **Elicitation** (spec-native, zero UI code; needs Claude Code ≥ 2.1.76)
   - Rich pickers, charts, live dashboards, scrollable lists → **MCP app widgets**
   - Neither → plain tool returning text/JSON

5. **What auth does the upstream service use?**
   - None / API key → straightforward
   - OAuth 2.0 → needs remote server with CIMD or DCR support

---

### Deployment model (Phase 2 of the builder)

Based on the answers, the builder recommends **one** path:

| Path | When to choose | Scaffolded by |
|---|---|---|
| **Remote streamable-HTTP** | Wrapping any cloud API — default recommendation | `build-mcp-server` inline |
| **MCP app** | Remote HTTP + UI widgets in chat | Routes to `build-mcp-app` |
| **MCPB** | Must run locally (local files, desktop app, OS APIs) | Routes to `build-mcpb` |
| **Local stdio** | Personal prototype only — painful to distribute | `build-mcp-server` inline, with MCPB upgrade note |

**Recommended stack:**
- TypeScript + `@modelcontextprotocol/sdk` — default; best spec coverage, first to get new features
- Python + `fastmcp` 3.x (`pip install fastmcp`) — if the user prefers Python or is wrapping a Python library

---

### What each path produces

**Remote HTTP** — the builder scaffolds:
- `package.json` with `@modelcontextprotocol/sdk`, `zod`, `express`
- `src/server.ts` using `McpServer` + `StreamableHTTPServerTransport` (stateless mode)
- Tool registrations via `server.registerTool(name, {description, inputSchema, annotations}, handler)`
- `annotations: { readOnlyHint: true }` on read-only tools (required for connector-directory submission)

**MCP app** — the builder routes to `build-mcp-app`, which adds:
- `@modelcontextprotocol/ext-apps` package (`registerAppTool`, `registerAppResource`, `RESOURCE_MIME_TYPE`)
- Each UI-enabled tool declares `_meta: { ui: { resourceUri: "ui://widgets/name.html" } }`
- Each resource is registered with `RESOURCE_MIME_TYPE = "text/html;profile=mcp-app"` (this tells the host to render it as an iframe widget, not display HTML source)
- Widget HTML inlines the ext-apps browser bundle via the `/*__EXT_APPS_BUNDLE__*/` placeholder — CDN script fetches are blocked by the iframe CSP, so bundling is mandatory
- Widget communicates with host via the `App` class: `app.ontoolresult`, `app.sendMessage`, `app.callServerTool`, `app.openLink` (use instead of `window.open` — blocked by sandbox)

**MCPB** — the builder routes to `build-mcpb`, which produces:
- A standard stdio MCP server (identical wire protocol — nothing MCPB-specific in tool logic)
- `manifest.json` with schema v0.4:
  ```json
  {
    "$schema": "https://raw.githubusercontent.com/anthropics/mcpb/main/schemas/mcpb-manifest-v0.4.schema.json",
    "manifest_version": "0.4",
    "server": {
      "type": "node",
      "entry_point": "server/index.js",
      "mcp_config": {
        "command": "node",
        "args": ["${__dirname}/server/index.js"],
        "env": { "ROOT_DIR": "${user_config.rootDir}" }
      }
    },
    "user_config": {
      "rootDir": { "type": "directory", "title": "Root directory", "required": true }
    }
  }
  ```
  - Use `${__dirname}` for bundle-relative paths
  - Use `${user_config.<key>}` to substitute install-time config values into env
  - **MCPB has no manifest-level sandbox** — the process runs with full user privileges; path validation and security are entirely the developer's responsibility
- Build pipeline using `@anthropic-ai/mcpb` CLI:
  ```bash
  npx @anthropic-ai/mcpb init       # interactive manifest creation
  npx @anthropic-ai/mcpb validate   # validate manifest against schema
  npx @anthropic-ai/mcpb pack       # zip and produce .mcpb file
  npx @anthropic-ai/mcpb sign dist/my-server.mcpb  # sign for distribution
  ```
  For Node: bundle with `esbuild --bundle --platform=node` before packing. For Python: vendor deps with `pip install -t server/vendor -r requirements.txt`.

---

### Testing (Phase 3 of the builder)

After scaffolding, the builder guides the user to test:

**All server types:**
```bash
npx @modelcontextprotocol/inspector node dist/index.js
```
Connect the running server to MCP Inspector for interactive tool testing.

**Remote HTTP and MCP app — add to Claude Code:**
```bash
claude mcp add <server-name> -- node $(pwd)/dist/index.js
```

**MCP app in Claude Desktop** — Claude Desktop requires a `command/args` config shape. Wrap with `mcp-remote` and force HTTP-only transport (prevents the SSE probe from swallowing widget-capability negotiation):
```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "http://localhost:3000/mcp", "--allow-http", "--transport", "http-only"]
    }
  }
}
```
After editing widget HTML, **fully quit Claude Desktop** (⌘Q / Alt+F4) and relaunch — Desktop caches UI resources aggressively and window-close is not enough.

**MCPB — validate and test before packing:**
```bash
npx @anthropic-ai/mcpb validate
npx @anthropic-ai/mcpb pack
# Install by dragging the .mcpb file onto Claude Desktop
```
Test on a machine **without** your dev toolchain before shipping — "works on my machine" failures in MCPB almost always trace to a dependency that wasn't bundled.

---

## Step 4 — After the server is built

Once the builder finishes, tell the user:

```
Your MCP server is ready. Next steps:

Remote HTTP / MCP app:
  npm run build
  Deploy to Cloudflare Workers, Railway, Fly.io, or any host
  Submit to the Anthropic connector directory:
  https://claude.com/docs/connectors/building/submission

MCPB:
  npx @anthropic-ai/mcpb sign dist/<name>.mcpb
  Distribute the .mcpb file (drag-to-install on Claude Desktop)

Optional — publish as a Claude plugin (adds skills, agents, commands):
  /plugin-to-mcp

Optional — generate MCP registry submission form:
  /mcp-submit-mcp-registry
```

---

## Routing directly to a sub-skill

If the user already knows what they need, skip `build-mcp-server` and go directly:

- UI widgets (forms, pickers, charts, live dashboards): `/mcp-server-dev:build-mcp-app`
- Bundled local server (`.mcpb`, no Node/Python for end users): `/mcp-server-dev:build-mcpb`
- Everything else (remote HTTP, local stdio, use-case discovery): `/mcp-server-dev:build-mcp-server`

---

## Common questions

**Remote HTTP vs MCPB vs local stdio?**
> - **Remote HTTP** — hosted server, zero install friction, one deploy for all users. Best for cloud APIs.
> - **MCPB** — local server bundled with runtime. No Node/Python needed. Best when the server must read local files or drive a desktop app. No manifest-level sandbox — security is your code's job.
> - **Local stdio** — easiest to prototype, hard to distribute. Personal tools only.

**Elicitation vs MCP app widgets?**
> Elicitation is spec-native (zero UI code) and covers simple yes/no, short enum picks, and flat forms. Use it if the host supports it (Claude Code ≥ 2.1.76). Reach for MCP app widgets when you need scrollable/searchable lists, visual previews, charts, or live-updating UI.

**Plugin install fails?**
> Check internet connection, confirm you are using Claude Code CLI (not the web app), run `claude plugin update` to refresh the marketplace index, then retry.
