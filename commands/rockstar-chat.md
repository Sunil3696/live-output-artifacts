---
description: Open a live Rockstar chat — a command-center chat artifact wired to the rockstar agent
---

Render a LIVE Cowork chat artifact wired to the `rockstar` orchestrator.

## Do exactly this
1. **Find the EXACT tool name.** In THIS session, find the available tool whose name ends
   in `__rockstar` (the orchestrator, NOT `rockstar_agents`) — it looks like
   `mcp__<serverId>__rockstar` where `<serverId>` is an opaque UUID
   (e.g. `mcp__5689d4cc-…__rockstar`). NOT the friendly `mcp__ir-mcp__rockstar`.
2. Read `skills/rockstar-chat/templates/rockstar-chat.html`.
3. Copy its contents as the HTML body, replacing `__ROCKSTAR_TOOL__` with that exact tool
   name (the `const TOOL` line). Change nothing else.
4. Call `mcp__cowork__create_artifact` with that body and `mcp_tools: ["<exact tool name>"]`.

## Why the tool name matters
In Cowork artifacts, MCP calls route by the UUID name `mcp__<serverId>__rockstar`, NOT the
friendly `mcp__ir-mcp__rockstar` — the friendly name fails inside Cowork before reaching the
server. The serverId changes per install, so read it live; never hardcode it.

The template is a full chat UI: it sends `{ user_input, session_id }` to the tool, keeps the
returned `session_id` for memory, shows a typing indicator, and renders markdown replies.
Do NOT write your own UI; do NOT paste these instructions into the page.
