# pod-dsh

The `dsh` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It installs the DeepSeek Harness (`dsh`) CLI and serves
its web UI as a supervised service.

## What it provides

Installs `@deepseek-ai/dsh` `0.1.7-rc.1` globally via npm and runs `dsh web`
under supervisord. The web app binds `127.0.0.1:3080` **only** (dsh-web-app
hard-rejects `--host 0.0.0.0` for RCE safety), so a charly-owned entrypoint
`dsh-entrypoint` runs the web app on loopback and exec's a `socat` forwarder on
the container's `eth0:3081`; the published port reaches the UI through socat.

The web UI authenticates every request with a random per-process launch token
(no flag disables it). The entrypoint captures the token from the readiness line
and persists it to `$DSH_HOME/web-token` on the `dsh` volume, so a fresh
`charly update` rebuild re-captures it. A tokenless `GET /` returns `401`.

| Property | Value |
|---|---|
| Service | `dsh` (`dsh-entrypoint`; `restart: always`) |
| Ports | `3080` (web UI, loopback only), `3081` (socat forwarder, published) |
| Requires | `layer-nodejs` (node ≥22.19.0), `layer-supervisord` |
| Volume | `dsh` at `~/.dsh` (profiles, plugin data, the captured web token) |
| Env | `DSH_HOME=~/.dsh` |
| Package | `socat` |

The repo also carries the `dsh-app` box (a minimal Fedora base composing
`nodejs` + the local `dsh` candy) and the `check-dsh-pod` disposable R10 bed.

## How to use it

Compose the candy into a box:

```yaml
my-dsh:
  candy:
    base: quay.io/fedora/fedora:43
    builder:
      npm: fedora-builder
    candy:
      - '@github.com/opencharly/layer-nodejs:<tag>'
      - '@github.com/opencharly/pod-dsh:<tag>'
```

```bash
charly box build my-dsh
charly start my-dsh
# open http://localhost:<published-port>
```

## The R10 bed

```bash
charly check run check-dsh-pod
```

The bed deploys `dsh-app` and asserts the CLI + node floor, the running `dsh`
service, the persisted web token, the token-authenticated UI in-box on
`127.0.0.1:3080` and through the socat forwarder on `127.0.0.1:3081`, the
published port's auth fence (`401` without a token), and the `dsh:` verb
dispatching in-box.

## Layout

- `charly.yml` — the `dsh:` candy entity, its `skill:` entity, the `dsh-app` box,
  and the `check-dsh-pod` bed.
- `package.json` — pins `@deepseek-ai/dsh` `0.1.7-rc.1`.
- `dsh-entrypoint` — the loopback web app + socat forwarder wrapper.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-tools:dsh` — the candy properties, the socat forwarding
  model, and the token-authentication flow.
- `/charly-coder:nodejs` — the required runtime dependency (node ≥22.19.0).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
