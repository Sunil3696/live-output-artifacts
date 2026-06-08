# live-output-artifacts

A Claude plugin that bundles your MCP server with a skill so that, whenever your
output-generating tool runs, Claude renders the result as a **live, re-fetching
Cowork artifact** instead of plain chat text.

## What's inside

```
live-output-artifacts/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # marketplace listing (for publishing)
├── .mcp.json                # ir-mcp server (remote HTTP)
├── skills/
│   └── render-live-artifact/
│       ├── SKILL.md         # drives the auto-artifact behavior
│       └── templates/
│           └── live-artifact.html
└── README.md
```

The **skill** is what makes the behavior automatic; the **plugin** packages the
skill + server together so it installs in one step. The plugin adds no new
capability beyond Cowork's existing `create_artifact` — it just makes the setup
repeatable and shippable.

## Already configured

- Server: `ir-mcp` at `https://paytest.testir.xyz/mcp` (`.mcp.json`).
- The skill triggers on **any** tool from the `ir-mcp` server, so new
  orchestrators you expose later are covered automatically — no per-tool edits.

## Ready to publish

Repo: `https://github.com/Sunil3696/live-output-artifacts`. Nothing left to fill
in. (In the artifact template, `TOOL` is a placeholder Claude substitutes with the
actual orchestrator name at artifact-creation time — you don't hardcode it.)

If your server uses SSE rather than streamable HTTP, change `"type": "http"` to
`"type": "sse"` in `.mcp.json`. (HTTP is recommended where supported.)

## Publish it

1. Push this folder to a GitHub repo (it can be both the plugin and the
   marketplace, since `marketplace.json` lists itself).
2. Validate locally: `claude plugin validate --plugin-dir ./live-output-artifacts`

## Install it (what your users run)

```
/plugin marketplace add Sunil3696/live-output-artifacts
/plugin install live-output-artifacts
/reload-plugins
```

After install, the MCP server connects and the skill activates automatically —
when any `ir-mcp` orchestrator produces output, Claude renders it as a live artifact.

## A note on "live" + LLM output

Live artifacts re-run your tool every time the page is opened. If your tool
regenerates LLM output on each call, the page will produce different results on
each reload. The skill handles this: if the output isn't meaningfully refreshable,
it falls back to a one-time static artifact. Decide which fits your tool.
