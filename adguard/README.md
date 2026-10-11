# adguard

Secondary DNS for the house. The primary is AdGuard on the Pi, `192.168.1.151`;
this one answers on `192.168.1.152` when the Pi does not.

| | |
|---|---|
| Admin UI | `http://192.168.1.152:3000`. Look, don't touch |
| Listens | 53, 3000, host networking |
| Config | Copied from the Pi by `adguard-sync`. Never edited here |
| Deploy | `ansible-playbook site.yml --limit bbox --tags adguard,stacks -e stack=adguard` |

No `.env`: nothing in this stack's compose file is templated.

## First deploy

1. The `adguard` role has to have run on bbox, or resolved still holds `:53`.
2. Deploy, then run the setup wizard at `http://192.168.1.152:3000`.
   - Admin interface: all interfaces, port **3000**. Caddy owns 80.
   - DNS server: all interfaces, port 53.
   - Login: the same username and password as the Pi's. `adguard-sync`
     uses one pair for both.
3. Deploy `adguard-sync`, then check the rewrites arrived:

```bash
dig @192.168.1.152 vault.samhudson.dev +short    # 192.168.1.151
dig @192.168.1.152 samhudson.dev +short          # 192.168.3.10
```

Only then add `.152` as the second DHCP name server on the UDM.

## One-way

The Pi is the origin. Rewrites, filters, clients, upstreams and blocked
services all flow Pi → bbox, so anything changed in this UI is overwritten on
the next sync. Query logs and stats are not synced; each instance keeps its
own, and clients split their queries between the two.

## What a Pi outage still breaks

Every service rewrite answers `.151`, the Pi's Caddy. With the Pi down this
instance still gives those answers, and the services behind them are still
unreachable. What survives is internet DNS, the `aqua-*` names (`.152`) and the
Minecraft apex (`pbox`).

## State

`conf/AdGuardHome.yaml` is a copy of the Pi's, so it needs no backup. `work/`
is this instance's own logs and stats.
