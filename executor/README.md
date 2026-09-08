# Executor

The missing integration layer for AI agents. Add an MCP server / OpenAPI spec /
GraphQL API once, give it credentials once, set per-tool policy once, then point
every MCP client at one endpoint.

- <https://github.com/UsefulSoftwareCo/executor>
- <https://executor.sh/docs>

Single container, no external services. Storage is a libSQL (SQLite) file under
`/data`; code execution runs in-process via QuickJS.

## Usage

```bash
cp env .env
docker compose up -d
```

Open <http://127.0.0.1:4788>. The first account created becomes the owner;
open registration is disabled afterwards, and further users join through
single-use invite links minted from the Admin page.

Connect a client:

```bash
npx add-mcp http://127.0.0.1:4788/mcp --transport http --name executor
```

Most MCP clients only load servers at startup, so restart the client or open a
new session before the tools show up.

## Behind a reverse proxy

Two things must change together, or browser logins get rejected:

1. `BIND_HOST=0.0.0.0` in `.env` so the proxy can reach the container.
2. `EXECUTOR_WEB_BASE_URL=https://your.domain` matching exactly what you type in
   the browser, including scheme and port.

To join the existing proxy network instead of publishing a port, append:

```yaml
networks:
  default:
    external: true
    name: nginx-proxy
```

## Health

Public, unauthenticated endpoints useful for probes and provisioning scripts:

- `GET /api/health`
- `GET /api/setup-status` — whether first-run setup is done

The image already declares its own `HEALTHCHECK`. It is distroless with no
shell, so any override in compose must use exec-form `CMD`, not `CMD-SHELL`.

## Backup

Everything is in the data directory (`./data` by default) — the database and the
generated session key. Stop the container before copying to avoid grabbing the
SQLite file mid-write:

```bash
docker compose stop
tar czf executor-$(date +%F).tar.gz data
docker compose start
```

## Notes

This instance stores every API credential you hand it, which makes it a
high-value target. It is also a single point of failure: if it is down, every
agent loses every tool at once. Keep it on loopback or behind authenticated
TLS, and leave `EXECUTOR_ALLOW_LOCAL_NETWORK` and `EXECUTOR_ALLOW_STDIO_MCP`
off unless you specifically need them — both widen what sandboxed or
user-configured code can reach on the host.
