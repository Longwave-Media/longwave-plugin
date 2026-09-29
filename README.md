# Longwave — the video pipeline for AI agents

**Give any AI agent a creator's whole video pipeline.** Turn one long-form episode into Shorts,
thumbnails, show notes and a podcast feed — then publish to the creator's **own** YouTube channel.

Works with Grok Bot, Grok Build, Cursor, Claude Code and Meta Muse. One MCP connection, OAuth 2.1,
and no API key anywhere.

**Agents orchestrate. Longwave renders and publishes.** The agent never downloads video, never holds
Google credentials, and never drives YouTube Studio.

---

## Why this exists

An agent can already cut video and drive a browser. Two things it **cannot** get for itself:

**1. A verified YouTube upload grant.** Google approved Longwave for `youtube.force-ssl` — production
mode, no user cap, no "unverified app" screen. The creator connects once, and Longwave delegates
scoped access to the agent. Standing up your own integration means OAuth app review and Google
verification before the first upload.

**2. A pipeline, not a command.** Cutting a Short is the easy part. Longwave also does the captions,
the title and description, the thumbnail, the playlist routing, the daily upload caps, the
cross-platform schedule, the podcast RSS feed and the retry when an upload fails halfway.

So this is not a wrapper around `yt-dlp` and a Studio bookmarklet. It is the publishing authority,
exposed as fifteen tools.

---

## Install

```
grok plugin install Longwave-Media/longwave-plugin
```

Or add it as a self-hosted catalog source:

```
grok plugin marketplace add Longwave-Media/longwave-plugin
```

Both work with **no review**. Cursor and Claude Code read the same files; Muse builds its own MCP
client from the URL.

## Connect

**Nothing to configure, and no credentials live in this repo.** An unauthenticated call to the MCP
endpoint returns:

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer resource_metadata="https://www.longwave.media/.well-known/oauth-protected-resource"
```

A conformant client follows that, registers itself, opens a browser, and the creator signs in and
approves. PKCE **S256** is required. This is deliberate: Agent Plugins v1.0.0 §7.2.1 forbids
embedding secrets in `headers`, and OAuth discovery is how the spec expects authorization to work.

- **MCP endpoint** — `https://www.longwave.media/api/mcp` (Streamable HTTP, stateless)
- **Protected-resource metadata** — `https://www.longwave.media/.well-known/oauth-protected-resource`
- **Authorization server** — `https://www.longwave.media/.well-known/oauth-authorization-server`

## Tools

Fifteen tools — enough to run a creator's clip pipeline end to end, and deliberately no more. The
surface is a **deny-by-default allowlist**, not a mirror of the API, so a tool exists here because it
has an ownership-checked backing route.

**Create and inspect** — granted on first connect

| Tool | Does |
|---|---|
| `account_status` | Channel connected? Credits left? Call this first. |
| `create_shorts_job` | Cut a long-form episode into Shorts. Free — credits are spent on publish. |
| `get_job` | Progress, clip count and errors for a job. |
| `list_clips` | What was produced, with per-platform status and live URLs. |
| `create_supercut` | Assemble one longer cut from a source video or from existing clips. |

**Read** — granted on first connect

| Tool | Does |
|---|---|
| `list_episodes` | The creator's long-form catalogue, newest first, with status. |
| `get_episode` | One episode in full: titles, notes, chapters, transcript, podcast state. |
| `get_insights` | Channel-level summary: episode counts, recurring topics, hook types. |
| `get_content_intelligence` | Longwave's written analysis of what to make next. |
| `get_channel_overview` | Connected platforms and how each is performing. |
| `list_playlists` | Playlists, and which one is the upload destination. |
| `list_thumbnail_styles` | The thumbnail style catalogue. |

**Change something public** — a separate approval, every time

| Tool | Does |
|---|---|
| `publish_shorts` | Publish Shorts to YouTube, with scheduling. |
| `reschedule_post` | Move a queued post to a different hour, or unschedule it. |
| `approve_thumbnail` | Publish an AI-generated thumbnail on the episode. |

Publishing and thumbnail changes are **not** granted on first connect — they are a step-up the
creator approves separately, because both change something public. The server asks for the extra
permission with `403` + `WWW-Authenticate: ... error="insufficient_scope"`.

`tools/list` returns the whole catalogue regardless of grant, so a client can see what exists and ask
for it; the grant is enforced on the call. **`tools/list` is authoritative** and carries each tool's
full JSON Schema — the tables above are for someone reading this repo.

Every mutating tool accepts an `idempotency_key`. Retry a timed-out call with the same key and you
get the original result rather than a second job or a second public post.

## What it costs

**The creator starts free: 200 credits on signup, no card, and they do not expire.**

Credits are charged to the creator **per Short that actually publishes** — nothing for a failed
render, nothing for a clip that is never posted.

| Action | Credits |
|---|---|
| Publish a Short | 1 |
| Publish a Short on autopilot | 1.5 |
| Publish to X through the shared app quota | 4 |
| Upload a full long-form episode | 5, plus 1 per GB |

**The agent never handles payment.** It holds no card and cannot reach a checkout. When the balance
runs low a tool returns a `top_up_url`; the agent's job is to hand it to the creator and stop.

## Where it publishes

- **Through the official API, by Longwave** — YouTube (the episode and its Shorts), X (a post and a
  supercut), a public Link-in-Bio hub, and podcast RSS syndicated to 13 directories.
- **As a downloadable file** — every Short is a vertical file on the CDN with its caption and
  metadata attached, so an agent can post it anywhere from the creator's own signed-in session.

Longwave is not a party to any third-party platform's terms and never contacts them. We describe the
capability; the creator's agent performs the action.

## Package layout

Two ecosystems describe plugin packages differently, and a client that finds no MCP config installs a
plugin with **zero tools**. Both layouts are therefore shipped:

| Path | Convention |
|---|---|
| `plugin.json` | [Agent Plugins v1.0.0](https://agent-plugins.org/specification) — closed schema, fixed location |
| `mcp.json` | Agent Plugins v1.0.0 — transport `streamable-http` |
| `.grok-plugin/plugin.json` | Grok / xAI marketplace — verified against xAI's own vendored `external_plugins/neon` and third-party `getsentry/plugin-grok` |
| `.mcp.json` | Grok MCP config — transport `http` |
| `skills/longwave/SKILL.md` | Shared by both — the instructions the model reads |
| `.grok-plugin/marketplace.json` | Self-hosted catalog (tracks repo HEAD) |
| `assets/logo.svg` | Listing logo — the Groundswell mark, monochrome `currentColor` per brand canon |

## Who maintains this

Longwave is a product of **Signal Group Limited** (New Zealand · NZBN 9429035771678 · Co. No 1396666).
The dashboard repo's `scripts/verify-plugin-package.ts` validates this package before any release —
58 checks covering both manifest conventions, author agreement, a single MCP endpoint across both
configs, and drift between the skill's `allowed-tools`, this README and the live tool manifest.

### Submitting to the official xAI catalog

Entries live in [`xai-org/plugin-marketplace`](https://github.com/xai-org/plugin-marketplace), where
the source must be **SHA-pinned to a full 40-character lowercase commit** —
`scripts/validate-catalog.py` enforces that, and Grok re-verifies `git rev-parse HEAD == sha` after
cloning, so a force-push cannot silently ship new code. xAI also expects the source to be an
**organisation** repository; ours is `Longwave-Media`.

```bash
git ls-remote https://github.com/Longwave-Media/longwave-plugin.git HEAD
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
agent receives only an opaque, scoped Longwave token.

## Support

`support@longwave.media` · <https://www.longwave.media> · [org profile](https://github.com/Longwave-Media)

## License

MIT for this package. That does **not** cover the hosted Longwave service.
