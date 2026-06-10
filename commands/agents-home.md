---
description: Build the Rockstar Agents home — a live artifact listing every agent with its skills and tools
---

Render the **Rockstar Agents home** as a LIVE Cowork artifact.

## Do exactly this — do not improvise the UI
1. **Find the EXACT tool name.** In THIS session, find the available tool whose name ends
   in `__rockstar_agents` — it looks like `mcp__<serverId>__rockstar_agents` where
   `<serverId>` is an opaque UUID (e.g. `mcp__5689d4cc-…__rockstar_agents`). Call it once
   to confirm it returns `{ agents: [...] }`.
2. Read the file `skills/rockstar-agents-home/templates/agents-home.html`.
3. Copy its contents as the HTML body, but **replace the token `__ROCKSTAR_AGENTS_TOOL__`
   with the exact tool name from step 1** (the `const TOOL` line). Change nothing else.
4. Call `mcp__cowork__create_artifact` with that HTML body and
   `mcp_tools: ["<exact tool name from step 1>"]` (required — the page can only call tools
   listed here).

## Tool-name rule — THIS is what was breaking it
In Cowork live artifacts, MCP calls route by the **fully-qualified UUID name**
`mcp__<serverId>__rockstar_agents`, NOT the friendly `mcp__ir-mcp__rockstar_agents`.
Using the friendly name makes the call fail INSIDE Cowork before it reaches the server
(no server log, artifact shows "Failed to load agents: Tool returned an error"). The
serverId UUID changes per install, so it must be read live every time — never hardcode it.

## Hard rules (the last attempt broke these)
- **Do NOT write your own HTML/CSS/layout.** Use the template as-is — it is already designed.
- **Do NOT paste any of these instructions, the procedure text, or any skill prose into the
  artifact.** The page must contain ONLY the template markup — no visible prompt/instruction text.
- **Do NOT print the catalog as chat text.** It MUST be one live artifact.
- **Do NOT hardcode the data** or add your own reload button — the template fetches live and the
  artifact header already has Reload.
- Use ONLY the real catalog data the tool returns; never invent agents, skills, or tools.
