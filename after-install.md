# Seltz is installed

Sign in before the first search. Use the same Hermes profile for configuration, sign-in, and chat.

## Sign in with Seltz (default)

The portable plugin format cannot declare OAuth. Add this entry to the active profile's
`config.yaml`, replacing any existing `mcp_servers.seltz` entry:

```yaml
mcp_servers:
  seltz:
    url: https://mcp.seltz.ai/mcp
    auth: oauth
    strict_redirect_headers: true
```

Then run `hermes mcp login seltz`. Hermes opens the Seltz Console. Pick an organization and
click Allow. Do not add an `Authorization` header, and do not set `SELTZ_API_KEY` for this path.

## API key (headless fallback)

Create a key in the [Seltz Console](https://console.seltz.ai/?utm_source=hermes&utm_medium=integration&utm_campaign=marketplace) and save it on the active profile:

```bash
hermes config set SELTZ_API_KEY sk_your_key
```

With no native `mcp_servers.seltz` entry, the packaged connection sends that key as
`Authorization: Bearer ${SELTZ_API_KEY}`.

Enable the plugin if it is still disabled: `hermes plugins enable seltz`. Start a new session
before asking Hermes to search.
