---
name: render-live-artifact
description: >
  Handler for talking to any ir-mcp orchestrator (rockstar, kane, head_of_*,
  mygentic_poc, and future ones). Invoke it UP FRONT and make the ir-mcp call
  THROUGH this skill, not as a bare tool call, so the whole exchange renders as UI:
  when the visualize show_widget tool is available, the orchestrator's questions
  become an inline form and its result an inline card in chat; otherwise (or when
  the user wants a durable page) render a LIVE Cowork artifact that re-fetches on
  reopen. TRIGGERS: "call rockstar", "rockstar", "talk to/ask/run rockstar",
  "call/run/ask an orchestrator by name", any request to use an ir-mcp tool, an
  orchestrator that asks a question / runs an interview / returns output, or "show
  this as a live
  page / make a live artifact / keep this refreshed". When an ir-mcp orchestrator is
  involved, invoke this skill.
---

# Render ir-mcp orchestrator interaction as UI

This applies to EVERY tool exposed by the `ir-mcp` server. There may be many
orchestrators now or in the future — do not hardcode a single tool name. Use
whichever ir-mcp tool you just called.

## When to invoke (read first)

Invoke this skill **up front**, the moment the user wants to reach an orchestrator
(e.g. "call rockstar") — do NOT call the ir-mcp tool as a bare tool call and print
its text. Make the call **through** this skill so both directions of the exchange
render as UI: the orchestrator's *asks* and its *returns*. If you have already
called an ir-mcp tool this turn, still invoke this skill now to render the result
rather than pasting it as plain text. A bare relay of orchestrator text (a raw
"reply with the number" list, etc.) means this skill was skipped — don't do that.

When an `ir-mcp` tool **asks the user something** or **returns output**, do NOT
just print raw text in chat. Render it as UI. There are two rendering modes; they
run **alongside** each other:

- **Inline-widget mode (in-chat, interactive).** When the visualize `show_widget`
  tool is available, render the orchestrator's *questions* as an inline form and its
  *result* as an inline card, driven by `sendPrompt` — the app-like feel of the
  standalone Rockstar skill. This is the default for the conversational
  back-and-forth. Full contract: **`references/inline-widgets.md`**.
- **Live Cowork artifact mode (durable side-panel page).** A reopenable page that
  re-fetches the orchestrator via `window.cowork.callMcpTool`. Use it when the user
  wants a durable/refreshable page, as the "Save as live page" target from an inline
  result, and as the fallback when `show_widget` is unavailable. Procedure below.

## Which mode? (decide once, early)

1. Detect inline capability ONCE per session: are `mcp__visualize__show_widget` and
   `mcp__visualize__read_me` in this session's tools?
2. **Orchestrator asks for input / runs an interview** → inline **ask** widget if
   capable (`references/inline-widgets.md` § When the orchestrator ASKS), else drive
   it in plain numbered text.
3. **Orchestrator returns a result** → inline **result** widget if capable
   (§ When the orchestrator RETURNS), with a "Save as live page" action that triggers
   the Cowork flow below. If not capable, or the user wants a durable/refreshing page,
   go straight to the live Cowork artifact.
4. Never block on the visualize tool — if it's missing, fall back cleanly.

## Live Cowork artifact — procedure

1. Call the relevant `ir-mcp` tool once normally and note the response shape and
   the exact tool name (e.g. `mcp__ir-mcp__<orchestrator>`).
2. Call `mcp__cowork__create_artifact` with:
   - An HTML body based on `templates/live-artifact.html`.
   - `mcp_tools` set to the SPECIFIC ir-mcp tool you just called. The page can
     only call tools listed here, so this is required for the live fetch. If the
     view should pull from several orchestrators, list each one.
3. In the page, set the `TOOL` constant to that tool name and fetch on load via
   `window.cowork.callMcpTool(TOOL, ARGS)`. Pass the SAME arguments you used in
   step 1 so the refreshed view matches.
4. Responses inside the artifact are wrapped — always unwrap with:
   `const data = r.structuredContent ?? JSON.parse(r.content[0].text);`
5. Render `data` by shape (orchestrator output is mixed):
   - **JSON object/array** → table or cards.
   - **Markdown/plain text** → escaped text block.
   - **HTML string** → inject into the container.
   The template already branches on these.
6. Do NOT hardcode the fetched result into the HTML, and do NOT add your own
   refresh button — the artifact header has a Reload button and reads are cached.

## Constraints

- CDN libraries allowed inside artifacts: Chart.js, Grid.js, Mermaid only;
  inline anything else.
- localStorage is fine for remembering the user's filter/sort choices, not for
  the data itself.
- Keep the page background transparent and avoid top-level padding.

## When NOT to use a live (re-fetching) artifact

If the orchestrator re-runs an LLM and produces different output on each call, a
live (re-fetching) artifact will regenerate on every reload — slow and
non-deterministic. This is exactly the conversational interview case: prefer
**inline-widget mode** for the back-and-forth, and when the user wants to keep the
result, render a one-time **static** Cowork artifact (hardcode the produced result)
rather than a re-fetching one, and tell the user why.

## Reference files

- `references/inline-widgets.md` — the inline-widget contract: capability detection,
  the natural-message conventions, the ask/result widgets, batching an interview, fallback.
  Read before your first inline widget of a session.
- `templates/inline-ask.html` — inline form for an orchestrator's questions.
- `templates/inline-result.html` — inline card for an orchestrator's result.
- `templates/live-artifact.html` — the durable live Cowork artifact.
