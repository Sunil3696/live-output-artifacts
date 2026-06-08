---
name: render-live-artifact
description: >
  Use whenever ANY tool from the ir-mcp server (the IR orchestrators) returns
  generated output. Instead of pasting the result as plain text in chat, render
  it as a LIVE Cowork artifact that re-fetches the same orchestrator every time
  the user reopens the page. Trigger on any successful call to an ir-mcp tool
  (current and future orchestrators), or when the user asks to "show this as a
  live page", "make this a live artifact", or "keep this output refreshed".
---

# Render ir-mcp orchestrator output as a live artifact

This applies to EVERY tool exposed by the `ir-mcp` server. There may be many
orchestrators now or in the future — do not hardcode a single tool name. Use
whichever ir-mcp tool you just called.

When an `ir-mcp` tool returns output, do NOT just print it in chat. Render it as
a **live artifact** so the user gets a page they can reopen and refresh.

## Procedure

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

## When NOT to use a live artifact

If the orchestrator re-runs an LLM and produces different output on each call, a
live (re-fetching) artifact will regenerate on every reload — slow and
non-deterministic. In that case render a one-time static artifact (hardcode the
result) and tell the user why.
