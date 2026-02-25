# DNS Services

This repo hosts a Docker Compose stack for a layered DNS resolver setup. Each service has a focused role (local overrides, caching, policy routing, and encrypted upstreams), wired together on a private bridge network.

## Resolution Flow

```
Client -> dnsmasq (host overrides) -> unbound (cache, serve-expired)
       -> dnsdist (policy routing + failover/fallback)
          -> Iran pool (default path for non-free domains)
          -> free path (dnscrypt -> sing-box SOCKS egress)
```

## Containers and Purpose

- **dnscrypt (gists/dnscrypt-proxy)**: Encrypts and forwards DNS queries (DNSCrypt/DoH/ODoH). Uses `dnscrypt/dnscrypt-proxy.toml`, `forwarding-rules.txt`, and `cloaking-rules.txt`.
- **sing-box (ghcr.io/sagernet/sing-box)**: SOCKS proxy used by dnscrypt for upstream egress. Configure via `sing-box/config.json` (see `config.json.example`).
- **dnsdist (powerdns/dnsdist-18)**: Policy/load-balancer in front of upstream resolvers. Loads domain policies from `dnsdist/ir-domains.txt` and `dnsdist/free-domains.txt`, routes by suffix/keyword rules, and can fallback from Iran pool responses to the free path on bad answers.
- **unbound (mvance/unbound)**: Caching resolver that forwards to dnsdist. Config in `unbound/local.conf` with `serve-expired` to keep serving responses during outages.
- **dnsmasq (jpillora/dnsmasq)**: Local resolver binding port 53 on the host, forwarding to unbound and honoring `/etc/hosts` overrides. Config in `dnsmasq/local.conf`.

## Project Layout

- `compose.yaml`: service definitions, network, ports.
- `dnscrypt/`: dnscrypt-proxy configuration and rules.
- `sing-box/`: proxy configuration (keep secrets out of git).
- `dnsdist/`: policy routing, upstream pools, and domain lists (`ir-domains.txt`, `free-domains.txt`).
- `unbound/`: caching resolver config.
- `dnsmasq/`: local resolver config and host entries (`hosts`).

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
- Keep large domain lists in `dnsdist/ir-domains.txt` updated carefully; dnsdist loads them at startup.
