# The stock caddy image has no DNS provider modules, so `tls { dns cloudflare }`
# fails with an unrecognised directive. DNS-01 needs a custom build.
#
# Same build as the Pi's caddy stack. Kept separate rather than shared because
# the two run on different hosts and neither should wait on the other.
FROM caddy:2-builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare

FROM caddy:2
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
