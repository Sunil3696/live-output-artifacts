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

Build a LIVE Cowork artifact that lists every Rockstar / IR agent the user can
access, where clicking an agent reveals the **skills** and **tools** inside it.

## Procedure
1. The data comes from the read-only `mcp__ir-mcp__rockstar_agents` tool. Call it
   once normally to confirm the shape:
   ```json
   { "agents": [
     { "name": "...", "displayName": "...", "description": "...",
       "skills": [ { "skillId": "...", "name": "...", "description": "..." } ],
       "tools":  [ { "name": "...", "description": "..." } ] }
   ] }
   ```
2. Call `mcp__cowork__create_artifact` with:
   - The HTML body from `templates/agents-home.html`.
   - `mcp_tools: ["mcp__ir-mcp__rockstar_agents"]` — required; the page can only
     call tools listed here.
3. The template already fetches on load via `window.cowork.callMcpTool`, unwraps
   the response, and renders clickable agent cards (click → skills + tools). Do
   NOT hardcode the data, and do NOT add your own reload button — the artifact
   header has one.

## Constraints
- Keep the page background transparent; avoid top-level padding.
- CDN libraries allowed inside artifacts: Chart.js, Grid.js, Mermaid only — inline
  everything else.
- Use ONLY the real catalog data the tool returns; never invent agents, skills, or tools.
- One "Rockstar Agents" home — update the existing artifact instead of duplicating.
