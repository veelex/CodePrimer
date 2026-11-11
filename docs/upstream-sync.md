# Staying in sync with upstream

CodePrimer is a remix of [CodeAssist](https://github.com/tyron12233/CodeAssist). Its value depends on
keeping up: upstream fixes compiler, dexer, indexing and build-engine bugs that this fork cannot find on
its own, and every month of drift makes the next merge worse.

## Where things stand

| | Commit | State |
| --- | --- | --- |
| Base this fork was cut from | `ee24e06` — upstream **3.34.0** (2026-10-29) | `REMIX.md` `Base:` line |
| `main` | base + 4 remix commits | Last merge 2026-11-11 |
| Upstream `main` | **3.44.0** (2026-11-10) | **297 commits ahead of the base** |
| `feature/zipcli` | base + upstream sync + ZipCLI work | 244 ahead / 4 behind `main`, unmerged |

The gap is the one real risk in this project. It is not yet painful — it is a one-sitting merge of
mostly-additive upstream work — and it gets materially worse every week it waits.

## Rules that keep the merge cheap

These are already how the remix is built; the point of writing them down is that they are load-bearing,
not stylistic.

1. **Never rename internal names.** `dev.ide.*` packages, Gradle module names, the `.caproj` format, the
   plugin SPI, and the `codeassist` color-scheme id stay as upstream has them. Renaming any of them turns
   every future merge into a rename conflict across hundreds of files.
2. **Change strings, not keys.** User-visible text says CodePrimer; `<string name="help_improve_codeassist">`
   keeps its key. Renaming keys invalidates every translation upstream adds.
3. **Keep remix edits isolated and small.** `applicationId`, `versionName`, the deep-link scheme, the
   blanked backend defaults, the `.github/workflows/apk.yml` additions. Anything bigger belongs upstream.
4. **Attribution stays.** Copyright headers, `LICENSE`, `LICENSE-EXCEPTION` are never touched.
5. **New work goes on a branch** (`feature/...`), and that branch absorbs upstream merges. `main` only
   receives: upstream syncs, and finished features.

## The merge, step by step

```sh
git remote add upstream https://github.com/tyron12233/CodeAssist.git   # once
git fetch upstream --tags
git switch -c sync/upstream-$(git describe --tags --abbrev=0 upstream/main)
git merge upstream/main                     # NOT rebase: the fork's history is public
```

Expect conflicts almost entirely in the six areas below. Each has one correct resolution — take
**upstream's** side, then re-apply the remix's line.

| Conflict area | Resolution |
| --- | --- |
| `app/ide-android/build.gradle.kts` (`applicationId`, `versionCode`, `versionName`) | Take upstream's structural changes; keep `io.github.veelex.codeprimer`, the `-cp.N` version name, and bump `versionCode` above the last release. |
| `app/ide-android/src/main/AndroidManifest.xml` | Upstream's changes; the deep-link scheme stays `codeprimer://`. |
| `.../composeResources/values*/strings.xml` | Upstream's new strings verbatim, then re-apply the CodePrimer wording. Fastest check: `grep -i codeassist` should only ever match **keys and comments**, never string values. |
| Any `.kt` with hardcoded "CodeAssist" (splash, toasts, notifications, `.caproj` `createdBy`, Learn content) | Upstream's code; CodePrimer's string. |
| `SUPABASE_URL` / `JUDGE_URL` / `STORE_MEDIA_URL` / AdMob ids in the build script | Keep upstream's new parameters; keep the **blank/test defaults**. |
| `.github/workflows/apk.yml` | This file is a remix addition (upstream has no APK job). Keep ours, re-read upstream's `ci.yml` for anything new it needs. |

Then:

```sh
./gradlew test                              # ~2,100 tests, all modules
./gradlew :ide-android:assembleProfile      # the variant releases actually ship
```

and install it on a device and compile one real project. A green build proves nothing here: R8 is off,
code is dexed at runtime, and the toolchain is loaded reflectively.

## Also check after every merge

- **Remix enforcement points still exist.** `grep -rn "io.github.veelex.codeprimer" --include=*.kts .`,
  `grep -rn "codeprimer://" --include=*.xml .` — upstream occasionally moves the places these live.
- **New user-visible strings** upstream added are translated into the CodePrimer name.
- **Upstream's own release notes** for anything that changes the merge strategy (new modules, plugin SPI
  version bumps, Gradle/AGP upgrades that change the wrapper or the version catalog).
- **Anything AI-authored** in this fork that touches the sandbox, agent tools, signing or file ops gets a
  line-by-line read — the `feature/zipcli` branch carries such commits and has never been reviewed.

## What this fork no longer has

`app/ide-desktop`, `app/ide-ios`, `app/build-cli`, `fastlane/`, `action.yml` and three workflows were
removed (see `REMIX.md`). At merge time, take **upstream's side for the rest of the tree and drop these
paths entirely** — do not try to keep them compiling. A delete/modify conflict on those paths is the
expected outcome, not a mistake:

```sh
git rm -r --cached app/ide-desktop app/ide-ios app/build-cli fastlane action.yml 2>/dev/null
git checkout --ours settings.gradle.kts .github/workflows/ci.yml   # then re-check the includes below
```

`settings.gradle.kts` is the only file where that rule inverts: it must keep **this fork's** `include`
list (three modules shorter) while taking upstream's additions for everything else.

## Cadence

Merge upstream **at each upstream release, within a week of it** — not when the app breaks. Before a
release of this fork, `main` must contain the newest upstream release or the release notes must say why
not:

```sh
git log --oneline main..upstream/main | wc -l     # must be 0 (or explained) before tagging
```

If a merge goes badly wrong, `git merge --abort` and do it in two steps: merge upstream into a clean
branch of the base commit first, then merge that into `main` — the remix diffs are small enough that the
second step is trivial.

## Why not just rebase the remix onto upstream?

Because the fork is public and GPL: rewriting `main` breaks every clone, every tag, and the provenance of
what was published. Four commits on top of a merge are cheap; a force-pushed history is not.
