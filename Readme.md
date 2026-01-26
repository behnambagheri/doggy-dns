# DNS Services

This repo hosts a Docker Compose stack for a layered DNS resolver setup. Each service has a focused role (local overrides, caching, policy routing, and encrypted upstreams), wired together on a private bridge network.

## Resolution Flow

```
Client -> dnsmasq (/etc/hosts overrides) -> unbound (cache, serve-expired)
       -> dnsdist (split .ir vs non-.ir, HA failover)
          -> Iran DNS pool (for .ir)
          -> dnscrypt (for other domains) -> sing-box (SOCKS egress)
```

## Containers and Purpose

- **dnscrypt (gists/dnscrypt-proxy)**: Encrypts and forwards DNS queries (DNSCrypt/DoH/ODoH). Uses `dnscrypt/dnscrypt-proxy.toml`, `forwarding-rules.txt`, and `cloaking-rules.txt`.
- **sing-box (ghcr.io/sagernet/sing-box)**: SOCKS proxy used by dnscrypt for upstream egress. Configure via `sing-box/config.json` (see `config.json.example`).
- **dnsdist (powerdns/dnsdist-18)**: Policy/load-balancer in front of upstream resolvers. Routes `.ir` domains to a dedicated pool and provides ordered failover via `dnsdist/dnsdist.conf` so if one Iran resolver fails, the next is tried.
- **unbound (mvance/unbound)**: Caching resolver that forwards to dnsdist. Config in `unbound/local.conf` with `serve-expired` to keep serving responses during outages.
- **dnsmasq (jpillora/dnsmasq)**: Local resolver binding port 53 on the host, forwarding to unbound and honoring `/etc/hosts` overrides. Config in `dnsmasq/local.conf`.

## Project Layout

- `docker-compose.yaml`: service definitions, network, ports.
- `dnscrypt/`: dnscrypt-proxy configuration and rules.
- `sing-box/`: proxy configuration (keep secrets out of git).
- `dnsdist/`: policy routing and upstream pools.
- `unbound/`: caching resolver config.
- `dnsmasq/`: local resolver config.

## Usage

Start the stack:

```sh
docker compose up -d
```

Check logs for a service:

```sh
docker compose logs -f dnscrypt
```

Restart a service after config changes:

```sh
docker compose restart dnsdist
```

Stop everything:

```sh
docker compose down
```

## Verify Resolution

Example query from the host:

```sh
dig myfriend.com @127.0.0.1
```

## Notes

- `dnsmasq` binds port 53 on the host; you may need elevated privileges.
- Create `sing-box/config.json` from `config.json.example` and keep credentials private.
- For blackout scenarios, tune cache retention in `unbound/local.conf` (e.g., `serve-expired-ttl`) to keep serving responses even when upstreams are unreachable.
