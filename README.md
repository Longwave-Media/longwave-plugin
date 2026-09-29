# Longwave agent plugin

The distributable package that lets an agent (Grok Bot, Grok Build, Cursor, Claude Code) cut
long-form episodes into YouTube Shorts and publish them through the creator's own Longwave
connection.

**Agents orchestrate. Longwave publishes.** The agent never downloads video and never drives
YouTube Studio; Longwave holds the creator's YouTube authorization on its own servers.

## Layout

```
longwave/
├── plugin.json                        # Agent Plugins v1.0.0 manifest (closed schema)
├── mcp.json                           # fixed discovery location — the MCP server declaration
├── skills/
│   └── longwave/SKILL.md              # what the model reads to decide when and how to act
├── .grok-plugin/marketplace.json      # self-hosted Grok catalog (tracks repo HEAD)
└── README.md
```

`plugin.json` and `mcp.json` are **fixed locations** — neither can be renamed or inlined. Both are
validated against the [Agent Plugins v1.0.0 schema](https://agent-plugins.org/specification).

## Before this package works, three things must be true

1. **This directory must be mirrored to a public repository under the company org.** The repo URL
   in `plugin.json`, `.grok-plugin/marketplace.json`, and the marketplace PR must all point at it.
   xAI rejects personal-fork sources for branded plugins as possible impersonation.
2. **`https://www.longwave.media/api/mcp` must be serving** the MCP endpoint (it is), with the
   OAuth 2.1 authorization server reachable at `/.well-known/oauth-authorization-server`.
3. **The `/.well-known/*` rewrites must be live in production.** They are configured in
   `next.config.mjs` but were never observed working on the dev server — see
   `docs/specs/AGENT_MCP_GATEWAY_SPEC.md` §5.7. If they 404, OAuth discovery fails and no connector
   can install.

## No credentials in this package

`mcp.json` declares the server and **nothing else**. Deliberately:

> "Header values are visible package data, not a portable secret mechanism. Plugins MUST NOT embed
> credentials or other secrets in `headers`." — Agent Plugins v1.0.0 §7.2.1

Authorization is client-managed OAuth discovery: an unauthenticated call to `/api/mcp` returns
`401` with `WWW-Authenticate: Bearer resource_metadata="…"`, and the client runs the browser consent
flow from there. There is no API key to paste, which is the point.

## Distribution — three paths, two with no review

| Path | Command | Review |
|---|---|---|
| Direct install | `grok plugin install Longwave-Media/longwave-grok-plugin` | none |
| Self-hosted catalog | `grok plugin marketplace add Longwave-Media/longwave-grok-plugin` | none |
| Official xAI catalog | PR to `xai-org/plugin-marketplace` | code-owner review |

### Submitting to the official catalog

Add one entry to `xai-org/plugin-marketplace`'s `.grok-plugin/marketplace.json` `plugins` array,
with the `source.sha` pinned to a **full 40-character commit SHA** (a moving ref would let a
force-push ship new code silently, which is why xAI requires the pin):

```json
{
  "name": "longwave",
  "description": "Turn long-form episodes into Shorts and publish them to the creator's own YouTube channel.",
  "category": "productivity",
  "source": {
    "source": "url",
    "url": "https://github.com/Longwave-Media/longwave-grok-plugin.git",
    "sha": "<40-char-commit-sha>"
  },
  "homepage": "https://www.longwave.media",
  "keywords": ["longwave", "youtube shorts", "shorts from podcast", "publish to youtube"],
  "domains": ["longwave.media", "www.longwave.media"]
}
```

Then regenerate their index and validate:

```bash
python3 scripts/generate-plugin-index.py
python3 scripts/validate-catalog.py
python3 scripts/generate-plugin-index.py --check
```

**To roll out an update**, bump the pinned `sha` in that same entry — never open a parallel entry.

**Keep `keywords` specific.** They drive Grok's plugin CTA; generic terms (`ai`, `cli`, `workflow`)
get pushed back because they mis-fire on unrelated requests.

## Muse and other MCP clients

Muse has no connector submission process, so there is nothing to submit. Any user can connect by
handing their Muse a public MCP URL. The paste-ready prompt is in
`docs/reference/AGENT_INTEGRATION.md`.

## License

The plugin package is MIT. That does **not** cover the hosted Longwave service.
