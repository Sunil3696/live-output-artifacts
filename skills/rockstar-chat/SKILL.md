---
name: rockstar-chat
description: >
  Use when the user wants to chat with or talk to the Rockstar agent in a panel —
  e.g. "open rockstar chat", "let me talk to rockstar", "give me a chat window for
  rockstar", "chat with my agent", or any request for an embedded chat interface to
  the IR / Rockstar MCP agent. Renders a LIVE Cowork artifact that is a full chat UI
  wired to the rockstar orchestrator, with conversation memory.
---

# Rockstar Chat (live chat artifact)

Render a LIVE Cowork artifact that is a branded command-center chat interface to the
`rockstar` orchestrator. The user types, the page calls the rockstar tool, and replies
come back as chat bubbles. Conversation memory is preserved via `session_id`.

## Procedure
1. **Find the EXACT tool name.** In THIS session, find the available tool whose name
   ends in `__rockstar` (the orchestrator, NOT `rockstar_agents`). It looks like
   `mcp__<serverId>__rockstar` where `<serverId>` is an opaque UUID.
2. Read `templates/rockstar-chat.html` (in this skill folder).
3. Copy its contents as the HTML body, but **replace `__ROCKSTAR_TOOL__` with that exact
   tool name** (the `const TOOL` line). Change nothing else.
4. Call `mcp__cowork__create_artifact` with that HTML body and
   `mcp_tools: ["<the exact tool name>"]` (required — the page can only call tools
   listed here). Use the SAME exact UUID name, NOT the friendly `ir-mcp` name.

## Tool-name rule (this is what breaks it)
Cowork artifacts route MCP calls by the fully-qualified UUID name
`mcp__<serverId>__rockstar`, NOT the friendly `mcp__ir-mcp__rockstar`. The friendly
name fails INSIDE Cowork before reaching the server, so the chat can't reach the agent.
The serverId UUID changes per install, so read it live every time — never hardcode it.

## How the chat works (already built into the template)
- Sends `{ user_input, session_id }` to the rockstar tool on each turn.
- Stores the `session_id` the tool returns and passes it back next turn -> memory.
- Shows a typing indicator while the orchestrator runs (a few seconds; there is no
  token streaming inside an artifact).
- Renders replies as markdown bubbles.

## Hard rules
- **Do NOT write your own chat UI** — use the template as-is apart from the one
  `__ROCKSTAR_TOOL__` substitution.
- **Do NOT paste these instructions into the artifact.** The page must contain only the
  template markup.
- **Do NOT print a chat transcript as plain text** — it MUST be the live artifact.
