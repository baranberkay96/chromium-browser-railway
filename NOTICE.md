# NOTICE: Chromium Cloud Browser template

## Base image and its components

The service is built `FROM lscr.io/linuxserver/chromium:7b8e918f-ls63@sha256:ff1b8235e20367c3b660d31892e14428687df75e2d62e274b7d8d40cf5537108` (built 2026-11-11). The image is used as published by linuxserver.io. This repository contains no copy of its source files.

| Component | Source | License | How it is used |
|---|---|---|---|
| linuxserver.io `chromium`, release `7b8e918f-ls63` | https://github.com/linuxserver/docker-chromium (git revision `c4bbd54b9bb6a505f0decb422214c39dfdae7650`) | GPL-3.0 | Base image (`FROM`), unmodified except for the build-time edits listed below |
| linuxserver.io baseimage-selkies, release `debiantrixie-794625c6-ls150` | https://github.com/linuxserver/docker-baseimage-selkies | GPL-3.0 | Layers of the base image: the s6 service definitions, `init-nginx`, `/defaults/default.conf`, the labwc Wayland session |
| Selkies | https://github.com/selkies-project/selkies | MPL-2.0 | Streaming server and web client, in the base image |
| s6-overlay 3.2.1.0 | https://github.com/just-containers/s6-overlay, https://skarnet.org/software/ | ISC | Process supervision. `railway-start` calls `s6-svscan`, `s6-rc-compile`, `s6-rc-init` and `s6-rc` from the image |
| catatonit | https://github.com/openSUSE/catatonit (distribution package in the base image) | GPL-2.0-or-later | Container init and subreaper (`ENTRYPOINT`) |
| Chromium 156.0.8078.79-1~deb13u1 and other Debian 13 packages (labwc, foot, nginx, PulseAudio, Mesa and others) | https://www.chromium.org/, https://packages.debian.org/trixie/chromium | Chromium: BSD-3-Clause with third-party components under their own licenses; other packages: each package's own license (`/usr/share/doc/*/copyright` in the image) | In the base image |

### GPL notes

- The GPL parts of the base image stay under their licenses. Their complete corresponding source is published by their authors at the repositories above, at the git revision named in the image label.
- **Modified files:** at build time, `services/browser/patch-image.sh` edits `/defaults/default.conf` (GPL-3.0, from docker-baseimage-selkies: basic auth switched on, an Origin check, anti-framing headers and a `/healthz` location) and `/etc/nginx/nginx.conf` (2 worker processes, error log on stderr). The script and the inserted snippets (`nginx-origin-map.conf`, `nginx-server-hardening.conf`) are the complete record of these changes. Anyone who distributes a built image of this template should keep them available with it.
- The files written for this template (`Dockerfile`, `railway-start.sh`, `pkg-restore.sh`, `patch-image.sh`, the two nginx snippets and the documentation) are offered under GPL-3.0-or-later.
- No source file of linuxserver.io, Selkies or s6-overlay was copied into this repository. `railway-start` re-implements, in its own code, the order of steps that s6-overlay's `preinit`, `stage0` and `rc.init` perform, and then calls the image's own s6 tools and service definitions.

## Upstream files checked

- `Dockerfile`, `root/defaults/autostart`, `root/usr/bin/wrapped-chromium` and `package_versions.txt` of docker-chromium at tag `7b8e918f-ls63`.
- `root/defaults/default.conf`, `root/etc/s6-overlay/s6-rc.d/init-nginx/run` and `root/defaults/startwm_wayland.sh` of docker-baseimage-selkies at tag `debiantrixie-794625c6-ls150` (the base of the pinned image: its 14 layers match the first 14 layers of the chromium image). Every anchor `patch-image.sh` edits is present there.

## Market templates

None were used. The "Microsoft Edge", "Edge" and "Google Chrome" browser templates on the Railway marketplace show the demand; their files and listings were not read or copied. Edge and Chrome are proprietary, so this template uses the open-source Chromium. It is derived from our own Ubuntu 26.04 XFCE desktop template.

Product names belong to their owners. Chrome and Chromium are trademarks of Google LLC; Edge is a trademark of Microsoft Corporation. This is a community template, not affiliated with Google, the Chromium project, Microsoft, linuxserver.io, the Selkies project or Railway.
