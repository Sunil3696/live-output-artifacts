---
description: Talk to a Rockstar / IR orchestrator (ir-mcp) and render the whole exchange as UI — inline widgets where available, a live Cowork page or plain text otherwise.
---

Drive a conversation with an **ir-mcp orchestrator** and render every step as UI by
following the **render-live-artifact** skill. Do NOT call the ir-mcp tool as a bare
tool call and paste its text — the exchange must render as UI in BOTH directions
(what the orchestrator asks, and what it returns).

Argument (optional): `$ARGUMENTS`
- Empty → use the `rockstar` orchestrator.
- A known orchestrator name (rockstar, kane, head_of_sales, head_of_delivery,
  mygentic_poc, …) → use that one.
- Anything else → treat it as the opening instruction/goal to send to `rockstar`.

## Steps

1. **Invoke the skill.** Load the `render-live-artifact` skill and follow
   `references/inline-widgets.md`. This command is just the guaranteed entry point;
   the skill holds the contract.
2. **Detect capability once.** Are `mcp__visualize__show_widget` and
   `mcp__visualize__read_me` in this session's tools?
   - **Available** → inline-widget mode. Call `read_me` with `{ modules:["interactive"] }`
     once, silently (never narrate it).
   - **Not available** → fall back: a live Cowork artifact if `mcp__cowork__create_artifact`
     exists, otherwise clean plain text (numbered lists for choices). Never block or stall.
3. **Find the tool.** Use the available ir-mcp tool whose name ends in
   `__<orchestrator>` (default `__rockstar`). Call it to start; keep the returned
   `ORCHESTRATOR_SESSION` id for the whole exchange.
4. **Render each turn as UI:**
   - Orchestrator **asks** a multi-question interview → render `templates/inline-ask.html`
     (batch every question into ONE form).
   - Orchestrator **asks you to pick one** (a client selector, a menu, any "reply with
     the number" list) → copy `templates/inline-choice.html` VERBATIM and fill one option
     row per choice (real name in both the label and the sendPrompt). Never surface a raw
     "reply with the number" list, and never hand-write the widget.
   - Copy template `<style>` blocks as-is — they use the fixed light palette. NEVER swap in
     host CSS variables (`--surface-*`, `--text-*`); that causes invisible white-on-white
     text and dark inputs.
   - Orchestrator **returns** a result → render `templates/inline-result.html`; its
     "Save as live page" action promotes the result to a live Cowork artifact.
5. **Thread the session — pass input straight through.** The widget sends exactly what's
   in it (the picked label, the answers as typed, the refine note, or a plain "Save as
   live page" / "Run again" / "Cancel"). Forward it to the orchestrator immediately with
   the session id re-attached — NO preamble, do not restate or rewrite the user's input,
   keep prose to one short line, then render the next widget. On "Save as live page",
   build the live Cowork artifact (skill's main flow). On "Cancel", stop cleanly. Read
   intent from the widget on screen, not from any prefix.

Keep chat prose to one line — the widget is the message. Strip orchestrator scaffolding
(`ORCHESTRATOR_SESSION`, "call the tool again…") from anything the user sees. If the
visualize tool is absent, do the same flow in the graceful fallback form.
