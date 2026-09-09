# SEC fork of hass-tailscale/hass-addons

Forked from https://github.com/hass-tailscale/hass-addons .

## Why
Upstream's `tailscale/Dockerfile` builds on `homeassistant/<arch>-base-debian:latest`,
which tracks **Debian bullseye**. Debian 11 LTS reached end-of-life early September 2026;
the `bullseye-security` `Release` file expired (last refresh 2026-08-31, Valid-Until
2026-09-07). Since then `apt update` in the image build fails (`Release file ... is
expired`, exit 100), so **every fresh build of the add-on fails** — which in our fleet
means failed restores, fresh provisions and replacement devices. Running instances keep
working (image already built); only new builds break.

## Change
`tailscale/Dockerfile`: base image `:latest` -> `:bookworm` (Debian 12, security-supported).
The Tailscale binary is a static Go binary fetched per-arch by `install.sh`, and the apt
packages (wget, iptables, iproute2, procps, iputils-ping) plus `iptables-nft` all exist on
bookworm, so this is a minimal, low-risk bump. The add-on **options/schema are unchanged**
(auth_key, login_server, hostname, userspace_networking, reset, force_reauth, ...) so the
standard SEC fleet config block and `ha authkey` keep working unchanged.

Add-on store repo slug (supervisor `sha1(url.lower())[:8]`): **908eaaec** ->
add-on slug `908eaaec_tailscale`.

## Resyncing with upstream
    git remote add upstream https://github.com/hass-tailscale/hass-addons
    git fetch upstream
    git merge upstream/main            # re-apply the :bookworm line if upstream touches Dockerfile
If upstream adopts a supported base image, this fork can be realigned or retired.
