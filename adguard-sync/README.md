# adguard-sync

Copies AdGuard config from the Pi to the secondary on this guest.
[adguardhome-sync](https://github.com/bakito/adguardhome-sync).

| | |
|---|---|
| Origin | `http://192.168.1.151:3000`, the Pi |
| Replica | `http://192.168.1.152:3000`, the `adguard` stack here |
| Interval | Every 10 minutes, and on start |
| Listens | Nothing. `API_PORT=0` |
| Deploy | `ansible-playbook site.yml --limit bbox --tags stacks -e stack=adguard-sync` |

## Syncs

Rewrites, filters and user rules, clients, blocked services, DNS server config,
access lists, general settings, query log and stats settings. Not the query log
or stats themselves, and not DHCP, which is turned off here because the UDM
does DHCP.

## Secrets

`ADGUARD_USERNAME` and `ADGUARD_PASSWORD`, the AdGuard admin login, used for
both instances. Changing the password on the Pi means changing it on bbox and
here too, or the sync fails.

## Checks

```bash
ssh bbox 'docker logs --tail 50 adguard-sync'    # "Sync done" per run
dig @192.168.1.152 vault.samhudson.dev +short    # 192.168.1.151
```

A rewrite added on the Pi should answer from `.152` within ten minutes.
`docker restart adguard-sync` forces a run.

## Versions

Pinned, and so is the replica's AdGuard. The sync talks to the API, so keep
both AdGuards on the same release and bump this when AdGuard moves.

## State

None.
