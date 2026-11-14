# Deploy and Host Chromium Cloud Browser with Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/PLACEHOLDER?referralCode=20H68q)

A full Chromium web browser that runs on Railway and streams into a tab of your own browser. It opens behind a generated password, and its profile (bookmarks, history, extensions, saved logins, downloads) lives on a persistent volume, so it survives redeploys. Chromium is the open-source browser that Google Chrome and Microsoft Edge are built on.

## About Hosting Chromium Cloud Browser

A browser in a tab needs a display session, a streaming server, a login in front of it and storage for the profile. This community template uses the linuxserver.io image `chromium` (pinned release `7b8e918f-ls63`, Chromium 156 from Debian 13), which streams the browser with Selkies over a single HTTPS WebSocket. Railway runs its own init as PID 1, which the image's s6-overlay init refuses to run under, so a small start script brings up the image's own services under s6 supervision instead. nginx asks for a login on every page except `/healthz`. `/config` (the home directory with the Chromium profile) is a Railway volume.

## Common Use Cases

- Use one persistent browser, with your bookmarks and extensions, from any computer, tablet or phone
- Test websites and web apps from a server in another region than your own
- Open unknown links and files in a remote browser instead of on your own device
- Keep long-running web sessions and dashboards open while your own computer is off

## Dependencies for Chromium Cloud Browser Hosting

- One service built from a small Dockerfile on top of `lscr.io/linuxserver/chromium` (no database, no external API keys)
- A Railway volume mounted at `/config` for the browser profile

### Deployment Dependencies

- [linuxserver/docker-chromium](https://github.com/linuxserver/docker-chromium)
- [linuxserver/docker-baseimage-selkies](https://github.com/linuxserver/docker-baseimage-selkies)
- [Selkies configuration reference](https://docs.linuxserver.io/selkies/user-guide/configuration/)
- [Selkies web client guide (sidebar, clipboard, files)](https://docs.linuxserver.io/selkies/user-guide/web-client/)
- [Chromium project](https://www.chromium.org/chromium-projects/)
- [Railway Fair Use Policy](https://railway.com/legal/fair-use)

### Implementation Details

| Service | What runs inside | Volume | Port |
|---|---|---|---|
| Browser | Chromium in a labwc Wayland session, D-Bus, audio, Selkies (loopback only), nginx (login, `/healthz`), all supervised by s6 | `/config` | 3000 (public HTTPS domain) |

**Plan:** RAM_PLACEHOLDER

**First login**

1. Wait for the deploy to turn healthy (the first build pulls a 1.1 GB image, so allow a few minutes).
2. Open the public URL. Your browser asks for a login.
3. User `chromium` (or your `CUSTOM_USER`; letters, digits and `. _ @ -` only, otherwise it stays `chromium`), password = `PASSWORD` from the service Variables tab.
4. Chromium appears and resizes to your window. The Selkies side menu has the clipboard box, file upload and download, audio, keyboard and display settings.
5. If you close Chromium, right-click the empty screen and choose Chromium to open it again.

**Start page and flags**

Set `CHROME_CLI` to a start URL and/or Chromium flags, for example `https://example.com --lang=de`, and redeploy.

**Security**

- The browser never runs without a login. Basic auth is on in nginx at build time, and if `PASSWORD` is empty, shorter than 8 characters or longer than 256 bytes, the login is locked with a random password and the deploy log says so (use 12 or more characters).
- Only a hash of the password is stored, and the password is removed from the environment before Chromium starts.
- Requests from other websites are refused (Origin check in nginx and Selkies), so a page you visit cannot use your cached login to drive the browser.
- Anyone with the password controls the browser and every account you are signed in to in it. The right-click menu also has a terminal, where the Linux user `abc` has passwordless sudo. The image's `HARDEN_DESKTOP=true` option disables sudo and terminals.
- Chromium runs with `--no-sandbox`, as in all container images of it: Railway containers cannot create the user namespaces Chromium's sandbox needs. Treat the whole container as the security boundary and avoid untrusted downloads.
- Browsers keep basic-auth logins until they close. Use a private window on shared computers.

**What persists**

Everything in `/config`: the Chromium profile (bookmarks, history, cookies, extensions, saved passwords, settings) and downloads. Packages installed with `sudo apt install` live in the image and reset on every redeploy. List them in `APT_PACKAGES` (for example `fonts-noto-cjk` for Chinese, Japanese and Korean pages) and they are reinstalled in the background after each start.

**No GPU: software rendering**

Railway has no GPUs. Pages are drawn on the CPU, and Selkies encodes the video stream on the CPU. Browsing, web apps and office tools run well; WebGL, 3D and high-frame-rate video are slow. Video playback costs CPU: lower the frame rate in the side menu, or set `SELKIES_FRAMERATE`, to save it.

**Shared memory**

Chromium keeps tab data in `/dev/shm`. The template sets `RAILWAY_SHM_SIZE_BYTES` to 1 GiB. If `/dev/shm` stays below 512 MiB, the start script adds `--disable-dev-shm-usage` so tabs do not crash.

**Audio and clipboard**

- Sound streams to your browser over the same WebSocket. Browsers only play it after you click in the page once.
- Text copy and paste works both ways. Chromium-based browsers ask for clipboard permission. In Firefox and Safari, use the clipboard box in the side menu.

**Fair use**

Use this browser for your own browsing and for testing websites. Railway's Fair Use Policy forbids operating proxies, VPNs or anonymization services, scraping that violates websites' terms, and abusive bots, as well as cryptocurrency mining and reselling compute. Do not share the login or offer the browser as a service to others. Websites may treat traffic from a data-center IP differently (extra captchas, region checks).

**Scaling and versions**

One user, one volume: run one replica and scale it vertically. The Chromium version is fixed by the image pin, so it does not update itself. To update (recommended monthly, for security fixes), change the `FROM` line in `services/browser/Dockerfile` to a newer release tag with its digest and redeploy. The profile is kept.

**Limits**

No WebRTC/UDP (the WebSocket stream is used instead), no Docker, and open tabs restart on every redeploy (Chromium offers to restore them). Railway's app sleeping stops the browser as well.

### Why Deploy Chromium Cloud Browser on Railway?

Railway gives the browser HTTPS, a persistent volume and usage-based billing without any server to manage. You get your own Chromium at a URL in a few minutes, pay for the RAM and CPU it actually uses, and can resize or redeploy it from the dashboard. This is a community template, not affiliated with Google, the Chromium project, linuxserver.io, the Selkies project or Railway.
