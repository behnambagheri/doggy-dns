# Repository Guidelines

## Project Structure & Module Organization
This repo contains Dockerized DNS service configs. Key paths:
- `compose.yaml`: service wiring, networks, and ports.
- `dnscrypt/`: dnscrypt-proxy config (`dnscrypt-proxy.toml`) plus `forwarding-rules.txt` and `cloaking-rules.txt`.
- `sing-box/`: proxy config; use `config.json` (see `config.json.example`).
- `dnsdist/`: dnsdist load-balancer policy in `dnsdist.conf`, plus `ir-domains.txt` and `free-domains.txt` routing lists.
- `unbound/`: caching resolver config in `local.conf`.
- `dnsmasq/`: local resolver config in `local.conf` and static host records in `hosts`.

## Build, Test, and Development Commands
This repo is configuration-only; services run via Docker Compose:
- `docker compose up -d`: start all DNS services in the background.
- `docker compose down`: stop and remove containers/networks.
- `docker compose logs -f dnsmasq`: follow logs for a specific service.
- `docker compose restart unbound`: restart a single service after config changes.

## Coding Style & Naming Conventions
- Indentation follows each format’s standard: 2 spaces for YAML, 4 spaces for Unbound `local.conf`, and consistent block indentation for TOML and dnsdist Lua.
- Keep config keys lowercase and group related settings (networking, caching, logging).
- Prefer descriptive names for upstreams/pools (e.g., `iran` pool in `dnsdist.conf`).

## Testing Guidelines
There is no automated test framework in this repo. Validate changes by:
- running `docker compose up -d`, then
- checking service health/logs (e.g., `docker compose logs -f dnsdist` and `docker compose logs -f dnscrypt`).

## Commit & Pull Request Guidelines
- Git history has no commits yet, so no established convention.
- Use concise, imperative commit subjects (e.g., “Add unbound cache tuning”).
- PRs should describe the change, affected service(s), and include sample logs or config snippets when behavior changes.

## Security & Configuration Tips
- `dnsmasq` binds port 53 on the host; expect elevated privileges on some systems.
- Keep `sing-box/config.json` out of version control if it includes credentials; derive it from `config.json.example`.
- `dnsdist/ir-domains.txt` is large; review list-source quality before bulk updates.
