# AGENTS.md — working on this repository

This file is for any AI coding agent (Grok Build, Cursor, Claude Code, DSH, Qwen) editing this repo.
Grok Build reads `AGENTS.md`; Cursor and Claude Code read their own rule files. The instructions are
the same.

## What this repository is

The **published distribution artifact** for Longwave's agent plugin. It is a manifest, an MCP config,
a skill file, a logo and documentation. **There is no application code here, and there must never be
a credential here.**

The service itself — the MCP endpoint, the OAuth 2.1 server, the fifteen tools and their backing
routes — lives in a **private** dashboard repository. This repo is the shop window, and the client
installs a **SHA-pinned commit** of it.

## The one rule that matters

**Nothing in this repository is the source of truth.** Every file is a copy, published from the
private repo's `plugins/longwave/` directory. Editing a file here without changing the original means
the next release silently reverts it.

If you are asked to change the tool list, the skill, or the manifests: make the change in the private
repo, run its validator, then re-publish here. Do not hand-edit the tool tables.

## Hard rules

- **Never add a secret.** Not a token, not a key, not a staging URL, not an internal hostname, not a
  customer name. The whole authentication design is OAuth *so that nothing is pasted*. A credential
  in this repo would be public the moment it is pushed, and the package is installed by strangers.
- **Never add, remove or rename a tool here** to change behaviour. The tool surface is a
  deny-by-default allowlist in the private repo's `lib/mcp/manifest.ts`. Changing it here changes a
  table, not the server — and creates drift that reads as a broken plugin.
- **Never change an endpoint URL** in `mcp.json` or `.mcp.json`. Both must point at
  `https://www.longwave.media/api/mcp`; the private repo's validator asserts they agree.
- **Never edit the tool tables by hand.** They are checked against the live manifest. If a tool is
  added, the table is regenerated in the private repo and copied here.
- **Never force-push.** The catalog entry pins a full 40-character SHA and Grok verifies
  `git rev-parse HEAD == sha` after cloning. A force-push invalidates the pin and the plugin fails to
  install. If you must rewrite history, re-pin the catalog entry in the same change.
- **Never `--no-verify`.**

## Before you push

The package is validated by the private repo's `scripts/verify-plugin-package.ts` (58 checks). Run it
from the private repo before copying anything here. It covers both manifest conventions, author
agreement across manifests, a single MCP endpoint across both MCP configs, and drift between the
skill's `allowed-tools`, the README's tool tables and the live manifest.

## The shape of the package

| Path | Purpose |
|---|---|
| `plugin.json`, `mcp.json` | Agent Plugins v1.0.0 — portable spec |
| `.grok-plugin/plugin.json`, `.mcp.json` | The Grok/xAI dotted convention |
| `skills/longwave/SKILL.md` | The instructions the model reads — the actual product surface |
| `.grok-plugin/marketplace.json` | Self-hosted catalog (tracks repo HEAD) |

Both conventions ship because a client that finds no MCP config installs a plugin with **zero
tools** — dead on arrival, and silent.

## Voice

Documentation here is read by creators, by reviewers at xAI, and by agents deciding whether to
recommend this plugin. Write in plain, specific English. **Describe capability; never instruct
platform automation.** The only automation guidance in `SKILL.md` is the prohibition
*"Never drive YouTube Studio."* Keep it that way.
