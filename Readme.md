# DNS Services

This repo hosts a Docker Compose stack for a layered DNS resolver setup. Each service has a focused role (local overrides, caching, policy routing, and encrypted upstreams), wired together on a private bridge network.

## Resolution Flow

```
Client -> dnsmasq (hosts overrides)
       -> dnsdist (policy routing + failover/fallback)
          -> free path (default: dnscrypt)
          -> Iran pool (.ir + listed domains, when pool is healthy)
```

## Containers and Purpose

- **dnscrypt (`docker.behnam.pro/gists/dnscrypt-proxy:war`)**: Encrypts and forwards DNS queries (DNSCrypt/DoH/ODoH). Uses `dnscrypt/dnscrypt-proxy.toml`, `forwarding-rules.txt`, and `cloaking-rules.txt`.
- **dnsdist (`docker.behnam.pro/powerdns/dnsdist-19:war`)**: Policy/load-balancer in front of upstream resolvers. Loads domain policies from `dnsdist/ir-domains.txt` and `dnsdist/free-domains.txt`, routes by suffix/keyword rules, defaults to free path, and only sends Iran-bound traffic to the `iran` pool when it is up.
- **dnsmasq (`docker.behnam.pro/jpillora/dnsmasq:war`)**: Local resolver binding port 53 on the host, forwarding to dnsdist and honoring local overrides from `dnsmasq/hosts`.
- **sing-box**: Present in `compose.yaml` but currently commented out.
- **unbound**: Present in `compose.yaml` and `unbound/local.conf` but currently commented out in active runtime flow.

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
- `dnsdist` currently uses `free` as the default route and falls back to free when Iran pool servers are unavailable.
- Keep large domain lists in `dnsdist/ir-domains.txt` updated carefully; dnsdist loads them at startup.
