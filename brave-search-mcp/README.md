# Brave Search MCP

MCP server for the Brave Search API: web, image, news, video, local business and
LLM-context search.

- <https://github.com/brave/brave-search-mcp-server>
- Image: <https://hub.docker.com/r/brave/brave-search-mcp-server>
- API key: <https://api-dashboard.search.brave.com/app/keys>

Uses `brave/brave-search-mcp-server`, published by Brave's own release CI to
Docker Hub and AWS Marketplace ECR. Not `mcp/brave-search` (Docker's MCP
Catalog build), which lags upstream and only ships a `latest` tag.

## Usage

```bash
cp env .env
# put your BRAVE_API_KEY in .env
docker compose up -d
```

Endpoint: `http://127.0.0.1:8081/mcp`

```bash
npx add-mcp http://127.0.0.1:8081/mcp --transport http --name brave-search
```

## Transport

Since v2.x the server defaults to stdio, so `BRAVE_MCP_TRANSPORT=http` is set
explicitly in the compose file. Drop it and the container still starts, but
nothing listens on 8080 and the published port answers nothing.

For stdio instead, clients run the image per-invocation rather than as a
long-lived service, and this compose file is not involved:

```bash
docker run -i --rm -e BRAVE_API_KEY brave/brave-search-mcp-server:v2.1.3
```

## Environment

- `BRAVE_API_KEY` — required
- `BRAVE_API_KEY_FILE` — path inside the container, for Docker secrets instead
  of an env var
- `BRAVE_MCP_TRANSPORT` — `http` or `stdio` (default `stdio`)
- `BRAVE_MCP_PORT` — default 8080
- `BRAVE_MCP_HOST` — default 0.0.0.0
- `BRAVE_MCP_LOG_LEVEL` — default `info`
- `BRAVE_MCP_STATELESS` — default `true`
- `BRAVE_MCP_ENABLED_TOOLS` / `BRAVE_MCP_DISABLED_TOOLS` — space-separated

## Hardening

`cap_drop: ALL`, `read_only: true` and `no-new-privileges` mirror the
`--cap-drop all --read-only` invocation in Brave's own release workflow. The
`/tmp` tmpfs is there so the read-only root filesystem has somewhere to write.

## Notes

The HTTP endpoint is unauthenticated: anyone who can reach the port can spend
your Brave API quota. Keep `BIND_HOST=127.0.0.1`, or put it behind a proxy that
authenticates, if the host is reachable from anywhere untrusted.

Pinned to `v2.1.3` in `env`. Each release also publishes full and short commit
SHA tags if you need a tighter pin.
