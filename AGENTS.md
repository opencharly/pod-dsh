# AGENTS.md — pod-dsh

Standalone candy repo for the `dsh` candy — the DeepSeek Harness CLI plus its
token-authenticated, socat-exposed web UI on `3080`/`3081`. The candy, its
`skill:` entity, the `dsh-app` box, and the `check-dsh-pod` R10 bed all live in
`charly.yml` at the repo root.

Canonical files:

- `charly.yml` — the `dsh:` candy entity (description, `require`, `port`,
  `volume`, `service`, `plan`), its `skill:` entity, the `dsh-app` box, and the
  `check-dsh-pod` bed.
- `package.json` — pins `@deepseek-ai/dsh` `0.1.7-rc.1` (installed globally by the
  npm builder).
- `dsh-entrypoint` — the loopback web app + `socat` forwarder wrapper.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-tools:dsh` — the owning skill: candy properties, the socat forwarding
  model, and the token-authentication flow. Load before editing, building,
  deploying, or troubleshooting this candy.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (the `dsh-app`
  box and the `check-dsh-pod` bed entities, tree-position nesting, sidecars).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs, the
  `check-dsh-pod` disposable bed, and `charly check run <bed>`.
- `/charly-image:image` / `/charly-image:layer` — the box / candy authoring
  reference (`charly.yml` schema, `plan:` step verbs, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is `charly check run check-dsh-pod`: it deploys the
  `dsh-app` box and asserts the CLI + node floor, the running service, the
  persisted web token, the token-authenticated UI on `127.0.0.1:3080` and through
  the socat forwarder on `127.0.0.1:3081`, the published port's `401` auth fence,
  and the `dsh:` verb dispatching in-box.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `dsh:` candy entity in `charly.yml`; the `skill:` entity in the same
  file is the owning skill's source — a candy change and its skill change land
  together.
- The npm version pin lives in `package.json`; the forwarding/token logic lives in
  `dsh-entrypoint`. Keep `port:` (`3081`), the `socat` package, and the
  entrypoint's listener in step — the published port maps to container `3081`,
  not `3080`.
- The `dsh` volume at `~/.dsh` is the persistent store (profiles, plugin data,
  `web-token`); a fresh `charly update` rebuild re-captures the token.
- The `skill:` entity is the source for `/charly-tools:dsh`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
