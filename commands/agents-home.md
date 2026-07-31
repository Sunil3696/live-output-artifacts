---
description: Build the Rockstar Agent Command Center — a live, per-user artifact for tracking, playbooks, a journey map, an AI roadmap, and every agent's skills and tools
---

Render the **Rockstar Agent Command Center** as a LIVE Cowork artifact.

## Do exactly this — do not improvise the UI
1. **Find the EXACT tool names — there are TWO.** In THIS session, find the available tools
   whose names end in `__rockstar_agents` and `__my_outputs` — they look like
   `mcp__<serverId>__rockstar_agents` and `mcp__<serverId>__my_outputs` (same UUID serverId).
   Call `rockstar_agents` once to confirm it returns `{ agents: [...] }`.
2. Read the file `skills/rockstar-agents-home/templates/agents-home.html`.
3. Copy its contents as the HTML body, then replace BOTH tokens (change nothing else):
   - `__ROCKSTAR_AGENTS_TOOL__` → the exact `__rockstar_agents` name (`const TOOL_AGENTS`)
   - `__MY_OUTPUTS_TOOL__` → the exact `__my_outputs` name (`const TOOL_OUTPUTS`)
4. Call `mcp__cowork__create_artifact` with that HTML body and
   `mcp_tools: ["<rockstar_agents name>", "<my_outputs name>"]` (required — BOTH; the page can
   only call tools listed here).

## What the artifact includes
The template renders a full command center in the **MyGentic brand** (paper · indigo,
Instrument Serif, MKG logo) with a top nav. The hero tab is the **AOS Map** — the company
as a live tech-tree built from the real skill catalog, laid out as an ordered
Foundation → Offer → Marketing → Lead Gen → Sales → Delivery → Ops & AI journey, scoped by a
**client selector** and driven by `my_outputs`: a node lights up **Shipped** when a
deliverable exists for that skill+client, with **Available / Locked** progression, a
**Next best action** banner, and a **Build / Run** toggle. **Business Dials** is the
first-run onboarding: a wizard auto-opens for new users (and stays as a tab) that asks goal +
horizon and the five dials (lead gen / sales / delivery / finance / revenue actual-vs-target),
derives a RAG rating, picks the one binding constraint, and builds a focused 3/5/7-skill sprint
of real catalog skills with Run buttons — then re-runs the dials on close with locked metrics
and keeps an immutable sprint history. It is **per-client** (same client selector as Outputs /
AOS Map), so each client gets its own dials, sprint and history (device-local, `cc_dials_v1`
keyed by client). The rest: **Dashboard**
(completion ring, AI "Your next move", activity heatmap + streak, Pinned, Continue),
**Playbooks** (guided multi-skill sequences), **Goal Roadmap** (type a goal, the AI builds
an ordered path), **All Skills** (search, filter, launch, favorites, per-skill notes + output
links, mark-done), and **Outputs** (every deliverable, filterable by client/skill/agent).
Personal usage is tracked **per user, device-local** (browser localStorage) — private to each
user. The AI panels use `window.cowork.askClaude` and fall back gracefully if it's
unavailable. Pass BOTH `__rockstar_agents` and `__my_outputs` in `mcp_tools`.

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
