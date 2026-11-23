# Seltz for Hermes Agent

![Seltz](assets/logo.svg)

Seltz gives Hermes source-backed results from the public web: general search, news, company profiles, LinkedIn and other public people profiles, financial reports, and Wikipedia. When you already have a URL, Seltz fetches the page as readable text.

New accounts receive [$20 in free request credit](https://docs.seltz.ai/pricing?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace).

- Product: [seltz.ai](https://seltz.ai/?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace)
- Console: [console.seltz.ai](https://console.seltz.ai/?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace)

## Example prompts

> Find the latest official guidance on EU AI Act transparency obligations and link the primary sources.

> Look up Stripe and its current CEO, including the CEO's public professional background.

> Fetch the European Commission's page on the AI Act and summarize the transparency obligations for providers.

## Tools

The hosted server exposes `search` and `fetch`. Hermes names them `mcp__seltz__search` and `mcp__seltz__fetch`. Both return Markdown for the agent to read. Parameter names below match the [Seltz MCP reference](https://docs.seltz.ai/integrations/mcp?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace). The live tool schema from the server is the authority if it adds a parameter or a tool.

### `search`

Searches Seltz's index. Each result includes its URL, a publication date when the source has one, snippets, and properties. Results do not include the full page, so call `fetch` on a URL when you need the page.

| Argument | Required | Use |
| --- | --- | --- |
| `query` | yes | The search query. |
| `scope` | no | Which dataset to search. Leave it unset for a general query. |
| `max_results` | no | How many results to return. The default is 10. The server lowers a higher value to 50. |
| `include_domains`, `exclude_domains` | no | Lists of up to 100 domains each. |
| `from_date`, `to_date` | no | Publication-date window, in UTC. A date (`2025-10-28`), a datetime (`2025-10-28T23:00:00Z`), or an offset such as `now-7d`. Not accepted for `companies` or `people`. |

Scopes, as documented for the server:

| Scope | What it covers |
| --- | --- |
| `web` | General web pages, including Wikipedia. The default. Currently in beta. |
| `news` | News articles. |
| `companies` | Public company profiles: name, industry, location, and company details. |
| `people` | Public professional profiles, including LinkedIn: name, role, company, and career history. |
| `financial-reports` | SEC filings from public companies (10-K, 10-Q, 8-K, and related forms), returned as searchable chunks. |
| `wikipedia` | Wikipedia articles. |

Scope details: [docs.seltz.ai/data/data-concepts](https://docs.seltz.ai/data/data-concepts?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace#scopes).

### `fetch`

Returns the full content of 1 to 5 pages as Markdown, with navigation, footers, and ads removed.

| Argument | Required | Use |
| --- | --- | --- |
| `urls` | yes | Absolute URLs. One to five, with no duplicates. |

`fetch` reads HTML and plain text. PDFs, images, and other binaries come back as `unsupported_content_type`. One call returns at most 100,000 characters of page text, split across the pages.

## Install

Requires Hermes Agent 0.22.2 or later, with its MCP support installed. Use the same Hermes profile for installation, configuration, sign-in, and chat.

```bash
hermes plugins install seltz --enable
```

You can also install this repository directly:

```bash
hermes plugins install seltz-ai/seltz-hermes-plugin --enable
```

Or ask Hermes:

> Install the Seltz plugin from the Hermes catalog and enable it.

Portable packages install disabled unless you pass `--enable`. If you skipped it:

```bash
hermes plugins enable seltz
```

If Hermes reports that the MCP SDK or HTTP transport is unavailable, install Hermes's MCP optional feature using the instructions for your Hermes version.

## Sign in (default)

The Agent Plugins manifest cannot declare `auth: oauth`. Browser sign-in is a native entry in the active profile's `config.yaml`. That entry replaces the packaged server of the same name. Replace any existing `mcp_servers.seltz` entry as a whole and keep your other servers. The full snippet is [examples/oauth.yaml](examples/oauth.yaml):

```yaml
mcp_servers:
  seltz:
    url: https://mcp.seltz.ai/mcp
    auth: oauth
    strict_redirect_headers: true
```

Then sign in from a terminal:

```bash
hermes mcp login seltz
```

Hermes discovers Seltz's OAuth server, opens the [Seltz Console](https://console.seltz.ai/?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace), and stores the tokens. Pick an organization and click Allow. The authorization scope the server advertises is `seltz:read`. You do not need `SELTZ_API_KEY` for this connection. Do not add an `Authorization` header: Hermes supplies the OAuth bearer token.

Cron jobs and other unattended runs cannot finish a browser sign-in. Sign in first, or use an API key for unattended work. If a refresh is rejected, run `hermes mcp login seltz` again.

Hermes documents the flow at [OAuth-authenticated HTTP servers](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp#oauth-authenticated-http-servers).

## API key (headless fallback)

Create a key in the [Seltz Console](https://console.seltz.ai/?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace). Seltz API keys use the `sk_` prefix. Save it on the active profile and keep it out of chat, URLs, and this repository:

```bash
hermes config set SELTZ_API_KEY sk_your_key
```

With no native `mcp_servers.seltz` entry, [mcp.json](mcp.json) sends the key as `Authorization: Bearer ${SELTZ_API_KEY}`. Hermes resolves the placeholder from the active profile.

To switch from OAuth to the key, replace the native entry as a whole so `auth: oauth` is removed. [examples/api-key.yaml](examples/api-key.yaml) is that entry. Leaving both `auth: oauth` and an `Authorization` header on the same entry is not a supported setup.

Start a new session after you sign in or set the key. Ask Hermes to find and load the `seltz` skill with `skills_list` and `skill_view`, then ask for current sources. Hermes prefixes skill names, so use `skills_list` for the full name.

## What this plugin does on your machine and the network

This package is a manifest, an MCP connection, one Markdown skill, and the Seltz logo. It runs no local code.

- Sends the queries you ask Seltz to search, and the page URLs you ask it to fetch, to `https://mcp.seltz.ai/mcp` over HTTPS.
- OAuth sign-in also contacts `https://console.seltz.ai`, which is the authorization server Seltz publishes for that resource. Hermes owns the browser flow, token storage, and refresh.
- The API-key fallback sends `Authorization: Bearer` with the profile's `SELTZ_API_KEY`. The plugin never reads credential files and never writes tokens.
- Runs no local process, shell command, background worker, or self-updater.
- The hosted server's tools can change independently of this package.

## Troubleshooting

**Sign-in.** If a tool call fails because the connection is not authenticated, run `hermes mcp login seltz` in the active profile. An organization admin can also block MCP access from the Console. Unattended runs should report the failure instead of waiting for a browser.

**API key.** If the header path rejects the key, confirm `SELTZ_API_KEY` is set on that profile and that the header is `Bearer` plus a space plus the key. Do not paste the key into chat. The placeholder below is not a credential:

```bash
hermes config set SELTZ_API_KEY sk_your_key
hermes mcp test seltz
```

**No credit.** If the server refuses the call because the organization is out of credit, add a payment method in the [Seltz Console](https://console.seltz.ai/?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace). Signing in again does not add credit.

## License

[Apache-2.0](LICENSE), the same license as [seltz-ai/seltz-codex-plugin](https://github.com/seltz-ai/seltz-codex-plugin).
