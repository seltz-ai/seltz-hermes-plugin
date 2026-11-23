# Catalog submission

Do not open this pull request until the plugin commit you intend to pin is on
`seltz-ai/seltz-hermes-plugin` and `hermes plugins validate --install-deps` has
passed at that commit. A Seltz owner or major contributor submits one file,
`plugin-catalog/seltz.yaml`, to `NousResearch/hermes-agent`. Nous maintainers
review and merge it. There is no self-serve listing.

Replace `<40-character-commit-sha>` with that commit. The pin must be the full
40-character SHA, not a branch, tag, or short SHA. Quote `version`. Leave
`tier` at `community` unless a Nous maintainer sets `verified`.

```yaml
name: seltz
title: Seltz
repo: https://github.com/seltz-ai/seltz-hermes-plugin
sha: "<40-character-commit-sha>"
description: "Source-backed web search, news, company and LinkedIn lookups, financial reports, Wikipedia, and page fetch. Sign in through the Seltz Console (OAuth); the packaged connection is the SELTZ_API_KEY fallback. Disclosure — Connects to https://mcp.seltz.ai/mcp and sends it the search queries and page URLs the agent asks Seltz to search or fetch. OAuth also contacts https://console.seltz.ai. Calls use the user's Seltz account."
maintainer: seltz-ai
tier: community
category: web
requires_hermes: ">=0.22.2"
docs_url: "https://docs.seltz.ai/integrations/mcp?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace"
version: "1.0.0"
platforms: []
image: https://raw.githubusercontent.com/seltz-ai/seltz-hermes-plugin/<40-character-commit-sha>/assets/seltz-hermes-banner.png
known_issues:
  - "Portable manifests cannot declare OAuth. Browser sign-in needs the native mcp_servers.seltz entry in the README. The packaged connection is the SELTZ_API_KEY fallback and is used only when that native entry is absent."
capabilities:
  provides_tools: []
  provides_hooks: []
  provides_middleware: []
  requires_env: [SELTZ_API_KEY]
```

`requires_env` lists `SELTZ_API_KEY` because `mcp.json` references it. OAuth
users do not need the key. The tool, hook, and middleware lists are empty
because the package registers no Python capabilities. Do not put remote MCP
tool names in `provides_tools`.

No `subdir`: the plugin is at the repository root.

The `image` URL must use the same SHA as `sha`, on `raw.githubusercontent.com`.
Hermes shows that banner at 2:1. The documented size is 1200×600; other aspect
ratios are centre-cropped. `assets/seltz-hermes-banner.png` is 2400×1200, the
same 2:1 ratio, so it is shown whole. It uses the official Seltz logo, the
line "for Hermes Agent", and the line "Source-backed web search · LinkedIn
profiles · page fetch". `assets/logo.svg` stays the wordmark in the README,
copied from `seltz-ai/seltz-codex-plugin`. `assets/seltz-catalog-card.svg` is
the previous wordmark-only card and is not the listing image.

## What to disclose in the catalog pull request

- Seltz-maintained portable package. The skill is adapted from the guidance in
  `seltz-ai/seltz-codex-plugin`. It is not a fork of a Hermes bundled plugin.
- HTTPS requests to `mcp.seltz.ai`. OAuth discovery and sign-in use
  `console.seltz.ai`, which is the authorization server published in Seltz's
  protected-resource metadata. The advertised scope is `seltz:read`.
- Recommended interactive setup is Hermes-managed OAuth via a native
  `mcp_servers.seltz` entry with `auth: oauth`. The packaged connection sends
  `Authorization: Bearer ${SELTZ_API_KEY}` only when that native entry is absent.
- No runtime dependencies, local executables, shell commands, background
  workers, credential-file reads, core overrides, or self-updater.
- One bundled skill. The hosted server's live tool schema is the authority.
  The public MCP page documents `search` (scopes `web`, `news`, `companies`,
  `people`, `financial-reports`, `wikipedia`) and `fetch`.
- No Desktop renderer code.

Admission rules and the entry schema are in
[plugin-catalog/README.md](https://github.com/NousResearch/hermes-agent/blob/main/plugin-catalog/README.md)
and [Submitting to the plugin catalog](https://hermes-agent.nousresearch.com/docs/developer-guide/plugins/catalog-submission).
The catalog CI job clones the pinned commit and runs
`hermes plugins validate --install-deps`, then checks for a self-updater and
for unscannable executable payloads.
