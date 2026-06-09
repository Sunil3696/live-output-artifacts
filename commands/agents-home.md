---
description: Build the Rockstar Agents home — a live artifact listing every agent with its skills and tools
---

Build a **LIVE Cowork artifact** (NOT plain chat text) that lists every Rockstar / IR agent
and lets the user click an agent to reveal its skills and tools.

Use the `rockstar-agents-home` skill's template at
`skills/rockstar-agents-home/templates/agents-home.html`:

1. Call `mcp__cowork__create_artifact` with that HTML body and
   `mcp_tools: ["mcp__ir-mcp__rockstar_agents"]` (required — the page can only call tools
   listed here).
2. Do NOT print the catalog as text in the chat. It MUST be rendered as a live artifact.
3. The template fetches on load via `window.cowork.callMcpTool("mcp__ir-mcp__rockstar_agents", {})`,
   unwraps the response, and renders clickable agent cards (click → skills + tools). It
   refreshes every time it opens. Do not add your own reload button.

Use ONLY the real catalog data; never invent agents, skills, or tools.
