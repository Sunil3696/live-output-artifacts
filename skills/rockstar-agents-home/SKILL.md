---
name: rockstar-agents-home
description: >
  Use when the user wants to browse or see the available Rockstar / IR agents —
  e.g. "show me the agents", "what agents do I have", "open the agents home",
  "list the orchestrators", "what can Rockstar do", "agent directory/menu". Renders
  a LIVE Cowork artifact listing every agent with the skills and tools inside each,
  re-fetching from the server every time the user reopens it.
---

# Rockstar Agents home (live)

Render a LIVE Cowork artifact that lists every Rockstar / IR agent the user can
access, with the skills and tools inside each. Each skill has a **Start chat**
button that opens a new Cowork chat for that skill.

## Procedure
1. **Find the EXACT tool name.** Look at the tools available in THIS session and find
   the one whose name ends in `__rockstar_agents`. It looks like
   `mcp__<serverId>__rockstar_agents` where `<serverId>` is an opaque UUID. Call it once
   to confirm it returns `{ agents: [...] }`. **This exact name is critical** — see the
   Tool-name rule below.
2. Read `templates/agents-home.html` (in this skill folder).
3. Copy that file's contents as the HTML body, but **replace the token
   `__ROCKSTAR_AGENTS_TOOL__` with the exact tool name from step 1** (the `const TOOL`
   line in the page). Do not change anything else in the markup.
4. Call `mcp__cowork__create_artifact` with that HTML body and
   `mcp_tools: ["<the exact tool name from step 1>"]` (required — the page can only call
   tools listed here). Use the SAME UUID name here, NOT the friendly `ir-mcp` name.
5. That's it. The template fetches live via `window.cowork.callMcpTool`, unwraps the
   response, renders the agent/skills/tools, has a search box, wires every Start chat
   button, and refreshes on reopen.

## Tool-name rule (this is what breaks if you get it wrong)
- In Cowork live artifacts, MCP tool calls route by the **fully-qualified UUID name**
  `mcp__<serverId>__rockstar_agents`, NOT the friendly connector name
  `mcp__ir-mcp__rockstar_agents`. Using the friendly name makes the call fail INSIDE
  Cowork before it reaches the server — the artifact shows "Failed to load agents:
  Tool returned an error". Always use the exact loaded tool name.

## Hard rules
- **Do NOT write your own HTML/CSS/layout** — the template is already designed; use it as-is
  apart from the one `__ROCKSTAR_AGENTS_TOOL__` substitution above.
- **Do NOT paste this skill's text, the procedure, or any instructions into the artifact.**
  The page must contain ONLY the template markup — no visible prompt text.
- **Do NOT print the catalog as chat text** — it MUST be one live artifact.
- **Do NOT hardcode the data** and do NOT add your own reload button (the header has one).
- Use ONLY the real catalog data the tool returns; never invent agents, skills, or tools.
- One "Rockstar Agents" home — update the existing artifact instead of duplicating.
