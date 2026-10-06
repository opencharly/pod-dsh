# pod-dsh

The DeepSeek Harness candy family of the OpenCharly candy library, as a
standalone repo (kind-prefixed naming). It installs the DeepSeek Harness
(`dsh`) CLI, serves its web UI as a supervised service, and ships the dsh-TUI
terminal-UI plugin as a typed terminal channel.

## What it provides

Three candies, split by concern:

| Candy | What it does |
|---|---|
| `dsh` | Installs `@deepseek-ai/dsh` `0.2.0-rc.2` globally via npm (the CLI only). |
| `dsh-web` | Composes `dsh`; serves the loopback web UI (`dsh web`) + a `socat` forwarder. |
| `dsh-tui` | Composes `dsh`; installs the dsh-TUI plugin into the `dsh-tui` profile + a `terminal:tmux` channel. |

### dsh-web

The web app binds `127.0.0.1:3080` **only** (dsh-web-app hard-rejects
`--host 0.0.0.0` for RCE safety), so a charly-owned entrypoint `dsh-entrypoint`
runs the web app on loopback and exec's a `socat` forwarder on the container's
`eth0:3081`; the published port reaches the UI through socat.

The web UI authenticates every request with a random per-process launch token
(no flag disables it). The entrypoint captures the token from the readiness line
and persists it to `$DSH_HOME/web-token` on the `dsh` volume, so a fresh
`charly update` rebuild re-captures it. A tokenless `GET /` returns `401`.

| Property | Value |
|---|---|
| Service | `dsh` (`dsh-entrypoint`; `restart: always`) |
| Ports | `3080` (web UI, loopback only), `3081` (socat forwarder, published) |
| Requires | `dsh`, `layer-nodejs` (node ≥22.19.0), `layer-supervisord` |
| Volume | `dsh` at `~/.dsh` (profiles, plugin data, the captured web token) |
| Env | `DSH_HOME=~/.dsh` |
| Package | `socat` |

### dsh-tui

`dsh-TUI` (https://github.com/ccch1mneyyy/dsh-TUI, npm
`@deepseek-harness-tui/dsh-tui`) is a **pure DSH plugin**, not a daemon: it is
added to the `dsh-tui` profile and launched by the shipped `dsh-tui` / `dst`
launcher. The candy publishes a `terminal_profile` (`dsh-tui`) and an
`agent_provide: tmux` channel, which is how a bed or a controller drives and
proves it (`charly agent terminal launch/snapshot/input/key/transcript`). The
first-run wizard is pre-recorded (`~/.dsh-tui/onboarding.json`) and the launchpad
disabled (`DSH_TUI_NO_LAUNCHPAD=1`) so the TUI lands on the chat screen
deterministically.

The repo also carries the `dsh-app` box (web UI) and the `dsh-tui-app` box
(terminal UI), each with its disposable R10 bed (`check-dsh-pod`,
`check-dsh-tui-pod`).

## How to use it

Compose a candy into a box:

```yaml
my-dsh-web:
  candy:
    base: quay.io/fedora/fedora:43
    builder:
      npm: fedora-builder
    candy:
      - '@github.com/opencharly/layer-nodejs:<tag>'
      - '@github.com/opencharly/pod-dsh/candy/dsh-web:<tag>'
```

```bash
charly box build my-dsh-web
charly start my-dsh-web
# open http://localhost:<published-port>
```

## The R10 beds

```bash
charly check run check-dsh-pod       # the web UI
charly check run check-dsh-tui-pod   # the terminal UI
```

`check-dsh-pod` deploys `dsh-app` and asserts the CLI + node floor, the running
`dsh` service, the persisted web token, the token-authenticated UI in-box on
`127.0.0.1:3080` and through the socat forwarder on `127.0.0.1:3081`, the
published port's auth fence (`401` without a token), and the `dsh:` verb
dispatching in-box.

`check-dsh-tui-pod` deploys `dsh-tui-app` and drives the TUI over the terminal
channel: it launches the `dsh-tui` profile, reads a structured virtual-screen
snapshot, types a numbered prompt whose answer is absent from the text, submits
it, asserts the live model answered, and reads durable transcript evidence.

## Layout

- `charly.yml` — the `skill:` entities and the `dsh-app` / `dsh-tui-app` boxes
  with their `check-dsh-pod` / `check-dsh-tui-pod` beds.
- `candy/dsh/` — the CLI candy + `package.json` (pins `@deepseek-ai/dsh`
  `0.2.0-rc.2`).
- `candy/dsh-web/` — the web candy + `dsh-entrypoint`.
- `candy/dsh-tui/` — the TUI candy + `package.json` (pins
  `@deepseek-harness-tui/dsh-tui` `0.13.0`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skills: `/charly-tools:dsh` (CLI + web) and `/charly-tools:dsh-tui`
  (terminal UI).
- `/charly-coder:nodejs` — the required runtime dependency (node ≥22.19.0).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
