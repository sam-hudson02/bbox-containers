# caddy (bbox)

The second TLS front door. Aqua Narrowboats only.

| | |
|---|---|
| Stack | `~/code/bbox-containers/caddy` → `~/containers/caddy` |
| Image | `caddy-cloudflare:local`, built from the stack's Dockerfile |
| Network | Host. Owns `:80` and `:443` on this guest |
| Data | `./data` (certificates, ACME account), `./config` |
| Secrets | `DOMAIN`, `CF_API_TOKEN`, `ACME_EMAIL` |

## Routes

| Name | To | |
|---|---|---|
| `aqua-cms.{$DOMAIN}` | `127.0.0.1:4322` | Keystatic, from the `aqua-cms` stack |
| `aqua-preview.{$DOMAIN}` | `127.0.0.1:4323` | Preview and the Publish page |
| `aqua-staging.{$DOMAIN}` | `192.168.3.10:8080` | The live site on `pbox` |

## Why a second Caddy

The Pi's Caddy serves every personal name on its `:443`. The Tailscale policy
filters by port, not by hostname, so a grant that let an Aqua editor reach the
CMS on the Pi would also reach Vaultwarden, AdGuard, CloudBeaver, Proxmox, DSM
and Home Assistant.

Two Caddies on two machines makes the split enforceable. bbox is its own
Tailscale node, shared into each editor's own tailnet, and the policy allows
its Tailscale address on `tcp:443` only. Editors reach the three names above
and nothing else; a shared node carries no subnet routes, so the LAN is not
visible to them. Adding an editor is a share invite, their login in the grant,
and a `basic_auth` line. It never touches the Pi.

## DNS

Each name has two answers. Inside the house, an AdGuard rewrite to
`192.168.1.152`. Everywhere else, a public Cloudflare A record to bbox's
Tailscale `100.x` address, DNS only, because sharees do not get this tailnet's
split DNS. A new route needs both.

## Basic auth

Tailscale membership is the real boundary. `basic_auth` is the second lock, and
it is also where the publish page gets the name it puts on each commit, so give
every editor their own line rather than sharing one.

```bash
docker exec caddy caddy hash-password        # prompts, prints the hash
```

Paste the result into the `(aqua-editors)` snippet in the Caddyfile and
redeploy. The username is what appears in the git history.

## Gotchas

Everything in the Pi's caddy note applies here too, since it is the same build
and the same DNS-01 setup. The ones that bite on this box specifically:

- **A changed Caddyfile does not restart Caddy.** Compose's config hash covers
  the compose file and `.env`, not a bind mount's contents. The `stacks` role
  restarts it; a hand edit on the guest does not.
- **Deploy caddy before the stack that needs the new route.**
- The staging route crosses into the DMZ. That works because trusted may open
  connections into the DMZ, and it is one-way: nothing on `pbox` can start a
  connection back.
- `aqua-cms` needs `request_body max_size 32MB` because Keystatic uploads
  photos. Without it an upload fails as a generic error. `aqua-staging` caps
  bodies at 1MB: its forms post text only and accept no files.

## Checks

```bash
ssh sam@192.168.1.152 'docker logs --tail 50 caddy'   # want: certificate obtained successfully
curl -sI https://aqua-cms.samhudson.dev | head -1     # 401 without credentials, which is correct
curl -sI -u sam:PASSWORD https://aqua-staging.samhudson.dev | head -1
```
