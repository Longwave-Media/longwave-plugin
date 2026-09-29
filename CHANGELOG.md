# Changelog

All notable changes to this package. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

This package is versioned independently of the hosted Longwave service. Because the client installs a
**SHA-pinned** commit, the version here is informational — the pin is what guarantees what a creator
gets.

## [1.0.0] — 2026-09-29

First release.

### Added

- **MCP server** at `https://www.longwave.media/api/mcp` — Streamable HTTP, stateless.
- **OAuth 2.1 authorization server with discovery** — RFC 9728 protected-resource metadata, RFC 8414
  authorization-server metadata, RFC 7591 dynamic client registration. PKCE S256 required. No API key
  is ever pasted.
- **Step-up authorization** for the two operations that change something public: publishing to a live
  channel and publishing a thumbnail. A fresh connection can read and create; it must ask for the
  rest, and the server answers `403` with `error="insufficient_scope"`.
- **Fifteen tools** across three grant tiers — create and inspect, read, and change-something-public.
- **Idempotency keys** on every mutating tool, so a timed-out call can be retried without creating a
  second job or a second public post.
- **Both plugin conventions shipped side by side** — Agent Plugins v1.0.0 (`plugin.json` +
  `mcp.json`) and the Grok/xAI dotted layout (`.grok-plugin/plugin.json` + `.mcp.json`). A client
  that finds no MCP config installs a plugin with zero tools, so shipping only one was not safe.
- **`SECURITY.md`**, this changelog, and an agent-facing `AGENTS.md` for anyone working on the repo.

### Known limitations

- Publishing is **YouTube and X**. TikTok and Instagram are reachable only as a downloaded file that
  the creator's agent posts from its own signed-in session.
- Episodes are ingested from a **YouTube URL on the creator's own channel**. Uploading a local file
  is not an agent-reachable path.
- The tool catalogue is deliberately **not** scope-filtered — see `SECURITY.md` for why.

[1.0.0]: https://github.com/Longwave-Media/longwave-plugin/releases/tag/v1.0.0
