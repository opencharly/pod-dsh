# Vendored dsh web-profile plugins — provenance

These four tarballs are the **prebuilt** community/native DSH plugins the
`dsh-web` web profile consumes as `file:` deps, so a profile install resolves
them offline — no `codeload.github.com` fetch, no per-dependency `npm install`
bootstrap (the repeated `ECONNRESET` flake `pod-dsh#26` recorded).

Each tarball is `npm pack` of the **opencharly fork** checkout at the pinned
commit below, after the fork's own `prepare`/`prepack` build ran (so the packed
`lib/` — and, for `dsh-workspace-enhancement`, the embedded `core/dist` Go
tarball — is the built artifact, not source).

## Per-tarball provenance

| Tarball (in this dir) | Source repo | Pinned commit | SHA-256 of the shipped tarball | `npm pack` reproduces byte-identical? |
|---|---|---|---|---|
| `perrylink-dsh-github-0.7.19.tgz` | `github.com/opencharly/dsh-github` (`fix/client-bundle`, PR #2) | `db37b9b6f81a72f884c362ce6f2c04d96bf6bad8` | `d36b994ed341f45f9e1bf44853e88422344cdc5cfe360e4d1c1bd2423f0ddbd5` | YES |
| `dsh-git-worktree-0.10.0.tgz` | `github.com/opencharly/dsh-git-worktree` (`master`) | `a910ffa0619954d951705a9832c1176890c4dc61` | `a56fd43d69c414bdbc7163998417c1bf29dc8dafd5baaea4a2ebfa7646258818` | YES |
| `dsh-workspace-enhancement-0.2.3.tgz` | `github.com/opencharly/dsh-workspace-enhancement` (`land/prepare`, PR #1) | `b57db61a18132ba0af275cabbc83eeb2a6f48f21` | `78d3681279ae03eaa98a0d443d1bf98ce6bb2cf9ff946b743249d91850f7f31d` | NO — see below |
| `dsh-opencharly-0.1.0.tgz` | `github.com/opencharly/dsh-opencharly` (`main`) | `f431b659405e86bc1d7530b3cedf01114eff3dcb` | `293acd7d7c1a7e8709ab661a3df16afa5473597a83357453df3f0f6ae26f35c3` | YES |

`SHA256SUMS` in this directory is the machine-checkable form of the four
hashes. `charly check run check-dsh-pod` verifies it in the running volume with
`sha256sum -c SHA256SUMS`, and asserts the seeded `package.json` names each
`file:./vendor-tgz/<tarball>` — so a revert to the `github:` git deps fails the
bed.

## The exact command

Run **in each fork checkout**, with the fork's `prepare`/`prepack` build enabled
(the default) so the packed artifact carries built `lib/`:

```bash
# per fork checkout, at the pinned commit:
git -C <fork> checkout <pinned-commit>
( cd <fork> && npm pack )         # emits <name>-<version>.tgz
sha256sum <name>-<version>.tgz
```

Example output for the one package whose `npm pack` prints a build line:

```
$ npm pack                         # in opencharly/dsh-git-worktree @ a910ffa0
> dsh-git-worktree@0.10.0 prepare
> pnpm run build
> dsh-git-worktree@0.10.0 build …/dsh-git-worktree
> node scripts/clean-lib.mjs && tsc && tsdown
ℹ [dsh-git-worktree-client] lib/client.js  1032.17 kB
✔ Build complete in 136ms
npm notice filename: dsh-git-worktree-0.10.0.tgz
npm notice package size: 645.1 kB
npm notice total files: 238
dsh-git-worktree-0.10.0.tgz
```

## Why `dsh-workspace-enhancement` is not byte-reproducible

Three of the four re-pack byte-identically from their pinned commit. The fourth
does not, and the cause was measured, not assumed:

- Its `prepare` runs `tsdown`, which compiles CSS Modules through
  `lightningcss`. The `cssExports` object it returns is iterated in a
  **non-deterministic order**, so the emitted `lib/client.js` differs between
  builds of the *same* commit (three consecutive `npm run build` runs produced
  three different `lib/client.js` hashes). Only `lib/client.js` differs —
  `lib/index.js` (plain `tsc`) is stable at
  `71594064d6aae4680ef22848c87f7893f70b1cee3acd36ae7d9d37fd123483c0`.
- Its embedded `core/dist/dsh-core-0.2.2-linux-x64.tar.gz` carries a **Go
  binary**, and Go builds are not byte-reproducible across toolchains **by the
  fork's own design** — it registers each built artifact's hash in
  `core/artifact.json` `compatHashes` instead of demanding reproducibility. The
  shipped binary's hash is registered there:
  `1531e1099b84a444761ce813c7f8b26c2ae9647d7525ea48a1924c0f9c5c1c40`.

So the verifiable contract for this tarball is its **committed blob hash**
(`78d36812…`, in `SHA256SUMS`) plus the inner core hash the fork itself
provenance-gates — not re-derivation by `npm pack`.

## Re-vendoring

To advance a plugin: check out the new fork commit, `npm pack` it, replace the
tarball here, update its row above and `SHA256SUMS`, then update the `file:`
specifier in `../package.json` (the version is in the tarball filename).
