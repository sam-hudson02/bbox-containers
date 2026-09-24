# immich-ml

Immich's machine learning service, running away from the Immich server.

| | |
|---|---|
| Endpoint | `http://192.168.1.152:3003` |
| Server | `immich-server` on the Pi, in [pi-containers](../../pi-containers) |
| Deploy | `ansible-playbook site.yml --limit bbox --tags stacks -e stack=immich-ml` |

## Why it is split

The Pi has 4 GB and also serves house DNS, so the Immich stack there runs with
machine learning omitted: smart search and face detection are off, and the
compose file says so. Immich's documented floor with machine learning is 6 GB.
This guest has the memory and the cores, and the split is supported upstream
rather than being a workaround.

## Joining the two halves

One setting, on the server, not here. In Immich: Administration → Settings →
Machine Learning Settings → Add URL, then `http://192.168.1.152:3003`.

Then turn smart search and face detection back on in the same settings page.
The Pi's compose comment warns that leaving them enabled against a service that
is not there queues jobs forever; the reverse is also true, and this endpoint
does nothing until the server is told about it.

Re-run Smart Search and Face Detection as missing-only jobs afterwards. The
library has never been indexed, so both have a full backlog to work through.

## No access to the library

The server posts the image preview it wants classified in the request body and
reads the result back. This container never opens the library on the Pi, so it
needs no share, no mount and no credentials. That is worth keeping true: it is
the reason this box can be rebuilt from nothing without touching the photos.

## Memory, and which model

The container's memory is almost entirely whichever CLIP model is selected in
the server's admin UI, and the choice is made there rather than here.

| Model | Resident |
|---|---|
| `ViT-B-32__openai`, the default | Around 1 GB |
| `ViT-B-16-SigLIP2__webli` | Around 3 GB |
| `ViT-gopt-16-SigLIP2-384__webli` | Around 6.6 GB |

The guest is capped at 6 GB with 2 GB of swap, which suits the default model
with room to try a mid-sized one. The largest will not fit while `pbox` holds
16 GB; that is a trade to make deliberately, not to discover during an index.

Leave `MACHINE_LEARNING_WORKERS` alone unless the cap goes up. Each worker
loads its own copy of the model, so two workers on the default model is 2 GB
before anything else.

## Exposure

Plain HTTP, no authentication of its own. Anything on trusted that can reach
port 3003 can ask this box to run inference on an image it supplies.

That is acceptable on trusted and would not be anywhere else. It is never port
forwarded, and the DMZ cannot reach trusted at all, so the exposed guests have
no path to it.
