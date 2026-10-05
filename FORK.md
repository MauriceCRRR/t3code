# Fork notes (`cursor-ui`)

Personal fork of [pingdotgg/t3code](https://github.com/pingdotgg/t3code) that restyles the
agent-chat UI. Upstream stays the source of truth for everything else.

## Rules

- UI only. Never edit `apps/server`, `packages/contracts`, `packages/shared`,
  `packages/client-runtime`, `packages/effect-*`, `native` or `apps/mobile`. The web client must
  keep working against a stock server.
- Fork code lives in `apps/web/src/cursor/`. Hooks into upstream files are one import plus one
  swap, marked `// cursor-ui:`.
- Replace, don't stack. When fork code supersedes upstream behaviour, the superseded code goes.
- Never run a packaged build (`dist:*`) before the app identity is changed; it would open the
  live `~/.t3/userdata` database. `vp run dev:desktop` is already isolated (bundle id
  `com.t3tools.t3code.dev.*`, profile `t3code-dev`, home `~/.t3/dev`).

## Branches

- `main`: mirror of `upstream/main`, fast-forward only.
- `cursor-ui`: the fork. Advances by merging the newest published nightly tag. Never rebase.
- GitHub Actions is turned off for the fork on purpose: upstream's workflows (release schedule, relay deploy) must never run here. Re-enable only for a fork-owned workflow.

One-time local config: `git config rerere.enabled true && git config rerere.autoupdate true && git config merge.conflictStyle zdiff3`

## Merging a nightly

```sh
git fetch upstream --tags --prune
TAG=$(git tag -l 'v*-nightly.*' | awk -F. '{print $NF, $0}' | sort -n | tail -1 | cut -d' ' -f2)
git switch main && git merge --ff-only upstream/main && git push origin main
git switch cursor-ui && git merge --no-ff -m "Merge upstream nightly $TAG" "$TAG"
git diff --quiet "$TAG" HEAD -- apps/server packages/contracts packages/shared packages/client-runtime  # must stay silent
vp run typecheck && vp run --filter @t3tools/web test
git push origin cursor-ui
```

## Baseline at the pinned tag (5 Oct 2026, this Mac)

Green: `vp run typecheck`, `vp run lint`, `vp run fmt:check`, `vp check`, `vp run knip:check`, web tests.
Server tests need a non-symlinked temp dir on macOS: `TMPDIR=/private/tmp/t3 vp run --filter t3 test`
(with the default `/var/folders` symlink, 7 path tests fail). Two `apps/mobile` native tests fail on
this Mac's SDK mix; they are macOS-only and outside this fork's scope.
