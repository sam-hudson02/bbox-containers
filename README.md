# bbox-containers

Compose stacks for `bbox`, the trusted services guest. Deployed by the `stacks`
role in `~/code/ansible`, which reads this directory on the control machine, so
no push is needed before deploying.

Secrets are SOPS-encrypted `.enc.env` files. Plaintext `.env` is generated at
deploy time and never committed.

## Stacks

| Stack | Port | Notes |
|---|---|---|
| `immich-ml` | 3003 | Immich's machine learning half. The server is on the Pi |
| `pricebuddy` | 8080 | Price tracker for the shopping list in the notes vault |
| `searxng` | 8081 | Metasearch front end |

## The guest

Unprivileged LXC `120` on the Proxmox host, Ubuntu 26.04, 4 cores, 6 GB of
memory and 2 GB of swap on a 32 GB root, untagged on the native trusted VLAN at
`192.168.1.152`. Docker runs inside it, which is why the container has
`features: nesting=1,keyctl=1`.

An LXC rather than a VM, unlike the DMZ guests. The reason those are VMs is
that they handle untrusted input where an escape would land on a hypervisor
sitting on trusted. This guest is already on trusted and handles nothing from
outside the house, so the container boundary is the right size of boundary and
the memory it does not use goes back to the host.

The name is reused. The old bbox was the VM at `192.168.1.190` that the network
rebuild retired, and this shares nothing with it. Its stack list is kept in
`~/code/ansible/docs/retired-bbox-vm-stacks.yml`, deliberately outside
`host_vars` so it cannot be deployed here by accident.

`192.168.1.190` is deliberately not reused either. The UDM still forwards the
Minecraft port to it, so a guest on that address would be answering the
internet until the forward is repointed.

## Networking

On trusted, so this guest can reach the Pi, the NAS and the Proxmox host, and
resolves through AdGuard on `192.168.1.151` like any other trusted host. That
is a real difference from the DMZ guests, which can reach none of them.

Nothing here is exposed to the internet and nothing here should be. Port 3003
has no authentication of its own; PriceBuddy on 8080 has its own login.

## What belongs here

Trusted-side services that want more compute than the Pi has. The Pi stays the
front door: it holds the TLS certificate, the resolver and the data. This box
takes the work that needs cores and memory, and is built so that losing it
costs a rebuild rather than a restore.
