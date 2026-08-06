# Inline widgets — render orchestrator I/O in chat

This is the contract for the skill's **inline-widget mode**: showing an ir-mcp
orchestrator's back-and-forth (what it **asks**, what it **returns**) as interactive
UI right inside the chat, using the `visualize` server's `show_widget` tool — the same
mechanism that makes the standalone Rockstar skill feel like an app.

It runs **alongside** the live Cowork artifact path (the rest of this skill), not
instead of it. Inline widgets carry the *conversation*; the Cowork artifact is the
*durable, reopenable page* the user promotes a result to.

## The mechanism (what's actually happening)

- **Inline UI** = HTML rendered in the chat stream by `show_widget` (visualize server).
- Buttons/forms call the global **`sendPrompt(text)`** — it posts a chat message as if
  the user typed it. That message re-enters this skill and you act on it (call the
  orchestrator, save a page, etc.). Navigation/editing inside a widget is plain
  client-side JS and costs **zero** model turns; only `sendPrompt` spends a turn.
- This is a different tool from `mcp__cowork__create_artifact`. Cowork artifacts live
  in a side panel and re-fetch via `window.cowork.callMcpTool`; inline widgets live in
  chat and drive the conversation via `sendPrompt`.

## Capability detection (once per session, before the first widget)

Inline mode requires the visualize inline-widget tools in THIS session:
`mcp__visualize__show_widget` **and** `mcp__visualize__read_me`.

- **Both present → inline mode.** Call `read_me` with `{ modules: ["interactive"] }`
  **once, silently** (never narrate it) before your first `show_widget`, then render
  widgets with `show_widget`.
- **Absent** (plain claude.ai, mobile, some hosts) → **fall back**: drive the interview
  in plain numbered text, and render returns the normal way (a live Cowork artifact per
  this skill's main flow, or a plain formatted answer). Never block on the tool.

## What the widgets send (exactly what's in the widget)

Widget controls fire `sendPrompt(...)` with the LITERAL content of the widget — the
row's own label, the text the user typed, or the button's own label — verbatim, with NO
prefix, wrapper, or rewriting (a rewritten message shows up in the chat and reads
machine-y, and Claude re-processes it at tool-call time anyway). When a widget is on
screen you already know what the message is a reply to:

| What the widget sends | What you do |
|---|---|
| the answers, one per line, in question order (exactly as typed) | map them to the questions you asked and thread them back to the SAME orchestrator session in ONE tool call |
| the picked option's label (e.g. "aruna") | that's the selection (e.g. the client) — continue with it |
| the typed name from the add-new field (e.g. "Foo Inc") | it isn't in the list → create/select that new entry, then continue |
| "Save as live page" | promote the last orchestrator result to a LIVE Cowork artifact (the main render-live-artifact flow) |
| "Run again" | re-run the same orchestrator with the same args |
| the refine note (exactly as typed) | continue the same session with this refinement |
| "Explain the strategy behind this." | explain the thinking/strategy behind the last result in plain terms (≤5-word-jargon rule); no tool call needed unless it helps |
| a question typed in the result card's "Ask a question" box | ANSWER IT FROM THE KNOWLEDGE BASE — call the ir-mcp knowledge tool (the one whose name ends in `__search_knowledge`) with the question, then render the answer (inline result if widgets are available). Don't guess from memory when the Brain can answer. |
| "That response was helpful." | thumbs-up feedback — call `record_feedback` with `rating:"up"` + context (see Feedback below), then acknowledge in ≤1 line |
| a "what was off" note, or "That response wasn't quite right." | thumbs-down feedback — call `record_feedback` with `rating:"down"` + the note, briefly acknowledge, and offer to fix (a refine/re-run) |
| "Skip" / "Cancel" | skip the current ask / abandon the interaction cleanly |

## Pass it straight through (no preamble, no rewriting)

When one of these widget messages arrives, **forward it to the orchestrator immediately**:

- Do NOT write a preamble ("Great, let me set that up…"), do NOT restate or paraphrase
  the user's input, and do NOT reformat their answers beyond mapping them to the
  questions the orchestrator asked. The user's words go to the tool as-is.
- You always hold the `ORCHESTRATOR_SESSION` id from the prior return, so the message
  never needs to carry it — you re-attach it on the call.
- Keep any chat prose to at most one short line; ideally just call the tool and render
  the next widget. The widget already showed the user what they chose.
- These are conventions, not magic strings: read intent from the on-screen widget +
  context, never from a prefix.

## When the orchestrator ASKS → `templates/inline-ask.html`

Orchestrators (rockstar, kane, head_of_*, mygentic_poc, …) often reply with a
multi-turn interview ("Question 1 of N"), a clarification, or a set of choices.

**Batch the interview (the default — do NOT go one question at a time):**
1. Detect the interview and its length N (usually "Question 1 of N").
2. Ask the orchestrator to list **all N questions at once**, numbered, in a single call.
3. Draft a best-effort answer for each from what you already know (so the user edits
   instead of composing from blank); leave blank when you have nothing.
4. Render `inline-ask.html` via `show_widget`: one field per question, pre-filled with
   the draft, a live `answered/N` progress bar, and Skip/Cancel. Repeat the `.ia-block`
   per question and fill every `{{...}}`.
5. On submit the widget sends the answers verbatim, one per line in question order. Map
   them to your questions and thread all answers to the orchestrator in ONE call → it
   jumps straight to producing its result. No preamble, no rewriting.

**Single free-text ask:** render `inline-ask.html` with one `.ia-block`.

**Choice ask (pick one — e.g. the client selector / "reply with the number" case):**
copy `templates/inline-choice.html` VERBATIM and fill it — do NOT hand-write the widget.
Repeat one `.mg-opt` row per option; put the option's REAL name as the visible label —
the script sends that label verbatim on click, so the message is exactly the name (never
the number — the number is UI, not data). Keep the "add new" input only if the
orchestrator allows a new entry (its Enter sends the typed name verbatim). The list is
already scrollable and capped. Never render a choice picker freehand — improvised markup
is where host-variable / invisible-text bugs creep in.

**Progress + a way out are mandatory** on any multi-step ask: the `answered/N` bar and a
Skip/Cancel control. A user mid-interview must never see a bare prompt with no sense of
"how far" and "how to get out".

## When the orchestrator RETURNS → `templates/inline-result.html`

Render the deliverable inline as a card. Fill `{{RESULT_BODY}}` by shape:
- **JSON object/array** → a compact table or key/value list (escape values).
- **Markdown / plain text** → an escaped `<pre>` block (or lightly formatted).
- **HTML string** → inject it directly into `.ir-body`.

The card carries a rating control (thumbs) in its header and this action row:
- **Save as live page** (primary) → build the durable LIVE Cowork artifact using this
  skill's main flow (`templates/live-artifact.html`, `create_artifact` with the right
  `mcp_tools`). Inline first; full page on demand.
- **Explain this strategy** → explain the thinking behind the result in plain terms.
  Teach, don't dump: no jargon without a five-word explanation. One short widget/answer.
- **Ask a question** → opens a box; the user's question is a **knowledge-base query**.
  Answer it by calling the ir-mcp knowledge tool (name ends `__search_knowledge`) and
  rendering the answer (a fresh inline result when widgets are available). This is the
  "ask a question to query the knowledge base" affordance — use the Brain, don't guess.
- **Run again** / **Refine** continue the same session.

**Thumbs feedback (response quality).** Thumbs up sends "That response was helpful.";
thumbs down opens an optional "what was off" note. On either, **persist it by calling the
ir-mcp feedback tool** (the one whose name ends in `__record_feedback`) — do NOT just
reply or write a memory note:

- `rating`: `"up"` for helpful, `"down"` otherwise.
- `note`: the user's "what was off" text (down only), verbatim.
- `orchestrator` / `skillId` / `skillName` / `clientId`: the source of the rated response
  — you hold these from the session (the orchestrator you ran, the active client, etc.).
- `outputId`: the deliverable's id if it came from `my_outputs`; `summary`: a ≤200-char
  snippet of what was rated.

Then acknowledge in ≤1 line. Thumbs down → also offer to fix it (refine or re-run). If
`record_feedback` isn't available (older server), fall back to a memory note. This is the
"start with thumbs up and down" loop — keep it lightweight, but the rating must land in
the server, not the chat.

Keep your chat prose to **one line** — the widget is the message.

## Design rules — match the command centre (MyGentic light theme)

Inline widgets use a **fixed light/paper palette** that matches the command centre
(`rockstar-agents-home`), NOT the host light/dark CSS variables. This keeps every
surface — the command centre, the ask form, the result card — reading as one branded
product, and makes the widget stand out cleanly as a light card in any chat. Both
templates already embed this; reuse the exact token block for any variant you build
(choice pickers, celebrations):

```css
--paper:#fff; --base:#f4f7fb; --raised:#eef2f8;
--bd:rgba(11,14,46,.10); --ink:#0b0e2e; --ink2:#242b5c; --muted:#4a5578; --faint:#8a93b0;
--accent:#3A4FB8; --violet:#8B7AE8; --success:#1f8f3a;
--f-disp:"Instrument Serif",Georgia,serif;      /* widget title */
--f-body:"Inter",-apple-system,system-ui,sans-serif;   /* body + fields */
--f-lbl:"Plus Jakarta Sans","Inter",sans-serif;        /* labels + buttons */
```

- **Outer card:** `background:var(--paper)`, `border:1px solid var(--bd)`,
  `border-radius:17px`, `box-shadow:0 8px 24px rgba(11,14,46,.06)`, `padding:18px 20px`.
- **Title:** `--f-disp` (Instrument Serif) ~19px; the orchestrator name in `--accent`.
  Labels/buttons in `--f-lbl`; body text and inputs in `--f-body`.
- **Primary button:** `background:var(--accent);color:#fff` (hover `#334499`).
  Secondary: white bg, `--bd` border, `--muted` text. Progress bar fill:
  `linear-gradient(90deg,var(--accent),var(--violet))`.
- **Fields:** `background:var(--base)`, `1px solid var(--bd)`, `border-radius:9px`;
  focus `border-color:var(--accent)` + `box-shadow:0 0 0 3px rgba(58,79,184,.12)`.
- Tabler outline icons (`ti ti-messages`, `ti-sparkles`, `ti-users`, `ti-send`,
  `ti-external-link`, `ti-refresh`, `ti-wand`), tinted `--accent`. Sentence case, no
  emoji, min font-size 11px. Load the three fonts once per widget via the Google Fonts
  `@import` at the top of `<style>` (CSP-allowed); the fallbacks cover a blocked load.
  Fill every `{{...}}` before rendering.

## Host form-control gotcha (read this)

The visualize host **force-styles native form controls** (`<button>`, `<input>`,
`<textarea>`, `<select>`) to its OWN light/dark theme, and those styles override your
classes. On a fixed light card in a dark host that means **button text goes invisible
and inputs render dark** — even though plain text (titles, labels in `<div>`/`<span>`)
renders fine. So:

- **Clickable buttons/options → styled `<div role="button" tabindex="0">`**, not
  `<button>`. Divs aren't themed by the host, so they honour the palette. Add a keydown
  handler so Enter/Space triggers the click (the templates already do). For a disabled
  look, toggle an `.is-off` class and guard the handler — divs have no `disabled`.
- **Real `<input>`/`<textarea>`** (you can't avoid these) → force every visual property
  with `!important`, and set BOTH `color` and `-webkit-text-fill-color` (WebKit uses the
  latter for inputs), plus `::placeholder` color `!important; opacity:1`.
- All three templates already do this — copy them as-is rather than reintroducing bare
  `<button>`/`<input>` styling.

## Don'ts

- **Never use host CSS variables** (`--surface-*`, `--text-*`, `--border`, `--radius`)
  in an inline widget. The widgets use the FIXED light palette (`--paper`, `--ink`,
  `--accent`, …) so they match the command centre. Host variables follow the host's
  dark mode and produce **white text on a white card (invisible)** and dark inputs —
  exactly the bug to avoid. Copy the template `<style>` as-is.
- **Never hand-write a widget** when a template exists (`inline-ask.html`,
  `inline-result.html`, `inline-choice.html`). Copy the file and fill its `{{...}}`.
- Don't show orchestrator scaffolding (`ORCHESTRATOR_SESSION`, "call the tool again…")
  inside a widget — relay only the real content.
- Don't paste this skill's text or these instructions into a widget.
- Don't render inline widgets when `show_widget` is unavailable — fall back instead.
- Don't call the orchestrator just to "check" state; only to enumerate, submit a batch,
  refine, or re-run. Widget editing is local and free.
