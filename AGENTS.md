# AGENTS.md — pod-dsh

Standalone candy repo for the DeepSeek Harness (`dsh`) capability — the CLI, the
token-authenticated socat-exposed web UI on `3080`/`3081`, and the dsh-TUI
terminal UI. Three candies split by concern (R3) live under each candy's own
directory:

- `candy/dsh/` — the `dsh:` CLI candy (`@deepseek-ai/dsh` via npm) + `package.json`.
- `candy/dsh-web/` — the `dsh-web:` candy: the loopback web service +
  `dsh-entrypoint` (composes `dsh`).
- `candy/dsh-tui/` — the `dsh-tui:` candy: the dsh-TUI plugin + a typed
  `terminal:tmux` channel (composes `dsh`) + `package.json`.

The `skill:` entities, the `dsh-app` (web) and `dsh-tui-app` (TUI) boxes, and
their R10 beds (`check-dsh-pod`, `check-dsh-tui-pod`) live in the root
`charly.yml`.

Canonical files:

- `charly.yml` — the `skill:` entities and the boxes + beds.
- `candy/dsh/package.json` — pins `@deepseek-ai/dsh` `0.2.0-rc.2`.
- `candy/dsh-web/dsh-entrypoint` — the loopback web app + `socat` forwarder wrapper.
- `candy/dsh-tui/package.json` — pins `@deepseek-harness-tui/dsh-tui` `0.13.0`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-tools:dsh` — the owning skill for the CLI + web candy family: candy
  properties, the socat forwarding model, and the token-authentication flow.
- `/charly-tools:dsh-tui` — the owning skill for the terminal UI candy: the
  plugin-is-not-a-daemon model and the terminal channel.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (the boxes and
  the beds, tree-position nesting, sidecars).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs, the
  disposable beds, and `charly check run <bed>`.
- `/charly-image:image` / `/charly-image:layer` — the box / candy authoring
  reference (`charly.yml` schema, `plan:` step verbs, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witnesses are `charly check run check-dsh-pod` (web UI) and
  `charly check run check-dsh-tui-pod` (terminal UI).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit a candy in its own directory; the `skill:` entity in `charly.yml` is the
  owning skill's source — a candy change and its skill change land together.
- The npm version pins live in `candy/dsh/package.json` and
  `candy/dsh-tui/package.json`; the forwarding/token logic lives in
  `candy/dsh-web/dsh-entrypoint`. Keep `port:` (`3081`), the `socat` package, and
  the entrypoint's listener in step — the published port maps to container
  `3081`, not `3080`.
- The `dsh` volume at `~/.dsh` is the persistent store (profiles, plugin data,
  `web-token`); a fresh `charly update` rebuild re-captures the token.
- A terminal channel is not a service: `dsh-tui-app` composes `check-keepalive` to
  hold the service-less pod at steady state.
- The `skill:` entities are the source for `/charly-tools:dsh` and
  `/charly-tools:dsh-tui`; never edit a generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
