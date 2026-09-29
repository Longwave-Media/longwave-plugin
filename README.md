# Longwave — agent plugin

Lets an agent (Grok Bot, Grok Build, Cursor, Claude Code, Muse) turn a creator's long-form episodes
into YouTube Shorts and publish them to that creator's **own** channel.

**Agents orchestrate. Longwave renders and publishes.** The agent never downloads video, never holds
Google credentials, and never drives YouTube Studio. Longwave holds the creator's YouTube
authorization on its own servers.

## Layout — both plugin conventions are shipped

Two ecosystems describe plugin packages differently, and a client that finds no MCP config installs a
plugin with **zero tools**. Both layouts are therefore present:

| Path | Convention |
|---|---|
| `plugin.json` | [Agent Plugins v1.0.0](https://agent-plugins.org/specification) — closed schema, fixed location |
| `mcp.json` | Agent Plugins v1.0.0 — transport `streamable-http` |
| `.grok-plugin/plugin.json` | Grok / xAI marketplace — verified against xAI's own vendored `external_plugins/neon` and third-party `getsentry/plugin-grok` |
| `.mcp.json` | Grok MCP config — transport `http` |
| `skills/longwave/SKILL.md` | Shared by both — the instructions the model reads |
| `.grok-plugin/marketplace.json` | Self-hosted catalog (tracks repo HEAD) |
| `assets/logo.svg` | Listing logo — the Groundswell mark, monochrome `currentColor` per brand canon |

The dashboard repo's `scripts/verify-plugin-package.ts` asserts that both manifests declare the same
name and author, and that both MCP configs point at the **same endpoint** — otherwise one loader
silently talks to the wrong place.

## Install

```
grok plugin install Longwave-Media/longwave-grok-plugin
```

Or add it as a self-hosted marketplace source:

```
grok plugin marketplace add Longwave-Media/longwave-grok-plugin
```

Both work with **no review**. Cursor and Claude Code read the same files; Muse builds its own client
from the MCP URL.

## Authentication

Nothing to configure, and **no credentials live in this repo.** An unauthenticated call to
`https://www.longwave.media/api/mcp` returns:

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer resource_metadata="https://www.longwave.media/.well-known/oauth-protected-resource"
```

A conformant client follows that, registers itself, opens a browser, and the creator signs in and
approves. PKCE S256 is required. This is deliberate: Agent Plugins v1.0.0 §7.2.1 forbids embedding
secrets in `headers`, and OAuth discovery is how the spec expects authorization to work.

## What the agent can do

Create Shorts from a long-form episode, poll progress, list what was made, publish (with scheduling),
create supercuts, reschedule a queued post, publish a thumbnail, and read channel insights, content
analysis, connected channels, playlists, episodes and the thumbnail style catalogue.

Publishing and thumbnail changes are **not** granted on first connect — they are a step-up the creator
approves separately, because both change something public. The server asks for the extra permission
with `403` + `WWW-Authenticate: ... error="insufficient_scope"`.

Credits are charged to the creator **per Short that actually publishes**. The agent never handles
payment; when the balance is low the tool returns a top-up link.

## Submitting to the official xAI catalog

Entries live in [`xai-org/plugin-marketplace`](https://github.com/xai-org/plugin-marketplace), where
the source must be **SHA-pinned to a full 40-character lowercase commit** — `scripts/validate-catalog.py`
enforces that, and Grok re-verifies `git rev-parse HEAD == sha` after cloning, so a force-push cannot
silently ship new code. xAI also expects the source to be an **organisation** repository; ours is
`Longwave-Media`.

Get the commit to pin:

```bash
git ls-remote https://github.com/Longwave-Media/longwave-grok-plugin.git HEAD
```

Then add the entry, regenerate the component index, validate, and open a PR:

```bash
python3 scripts/generate-plugin-index.py
python3 scripts/validate-catalog.py
```

**Re-pin `sha` on every release** — never open a parallel entry.

## Privacy and data

This package collects nothing and stores nothing. Anything the creator acts on is handled by the
hosted Longwave service under its own terms: <https://www.longwave.media/terms> and
<https://www.longwave.media/privacy>. YouTube credentials never leave Longwave's servers, and the
agent receives only an opaque Longwave token.

## Support

`support@longwave.media` · <https://www.longwave.media>

## License

MIT for this package. That does **not** cover the hosted Longwave service.
