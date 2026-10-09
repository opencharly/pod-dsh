# Vendored dsh web-profile plugins — provenance

These two tarballs are the **prebuilt** community/native DSH plugins the
`dsh-web` web profile consumes as `file:` deps, so a profile install resolves
them offline — no `codeload.github.com` fetch, no per-dependency `npm install`
bootstrap (the repeated `ECONNRESET` flake `pod-dsh#26` recorded).

Each tarball is `npm pack` of the **opencharly fork** checkout at the pinned
commit below, after the fork's own `prepare`/`prepack` build ran (so the packed
`lib/` is the built artifact, not source).

`@perrylink/dsh-github` and `dsh-workspace-enhancement` were dropped from this
profile on 2026-10-09 (see `CHANGELOG/`); their tarballs, rows and hashes are
gone with them.

## Per-tarball provenance

| Tarball (in this dir) | Source repo | Pinned commit | SHA-256 of the shipped tarball | `npm pack` reproduces byte-identical? |
|---|---|---|---|---|
| `dsh-git-worktree-0.10.0.tgz` | `github.com/opencharly/dsh-git-worktree` (`master`) | `a910ffa0619954d951705a9832c1176890c4dc61` | `a56fd43d69c414bdbc7163998417c1bf29dc8dafd5baaea4a2ebfa7646258818` | YES |
| `dsh-opencharly-0.1.0.tgz` | `github.com/opencharly/dsh-opencharly` (`main`) | `f431b659405e86bc1d7530b3cedf01114eff3dcb` | `293acd7d7c1a7e8709ab661a3df16afa5473597a83357453df3f0f6ae26f35c3` | YES |

`SHA256SUMS` in this directory is the machine-checkable form of the two
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

## Re-vendoring

To advance a plugin: check out the new fork commit, `npm pack` it, replace the
tarball here, update its row above and `SHA256SUMS`, then update the `file:`
specifier in `../package.json` (the version is in the tarball filename).
