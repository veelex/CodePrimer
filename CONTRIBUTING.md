# Contributing to CodePrimer

CodePrimer is a remix of [CodeAssist](https://github.com/tyron12233/CodeAssist) by tyron12233
(GPL-3.0-or-later). Bug reports, build fixes and features that suit an on-device IDE are welcome.

Because this is a remix, the first question for any large change is **"does this belong upstream?"**
Anything that is not CodePrimer-specific should be written so it can be sent to
[tyron12233/CodeAssist](https://github.com/tyron12233/CodeAssist) instead. See
[`docs/upstream-sync.md`](docs/upstream-sync.md) for the merge rules that keep that possible.

## Requirements

- **JDK 25** (what CI uses)
- **Android SDK**: `platforms;android-36`, `platforms;android-36.1`, `build-tools;36.0.0`
- ~8 GB free disk for a full build; the `minified` variant wants ~8 GB of Gradle heap on top

## Getting the source

The repository commits its own toolchain binaries — `app/ide-android/src/main/sdk/android.jar` (~28 MB)
and four `libaapt2.so` (~18 MB) — so a full clone is heavier than the source suggests. On a slow or
metered link:

```sh
git clone --depth 1 --single-branch https://github.com/veelex/CodePrimer.git
```

and fetch more history only if you need `git log`/`git blame` (`git fetch --deepen=200`).

## Build and test

| Command | What it produces |
| --- | --- |
| `./gradlew :ide-android:assembleDebug` | Debuggable APK, debug key. Installs directly. |
| `./gradlew :ide-android:assembleProfile` | **Release-like, non-debuggable**, installs with the debug key. Use this to judge runtime/typing/Compose performance — a debuggable build runs with ART optimizations off. Also what the release workflow ships. |
| `./gradlew :ide-android:assembleRelease` | Shipping build. Unsigned unless `keystore.properties` is present (see `keystore.properties.example`). R8 stays off by design: JDT/D8/apksig are loaded reflectively and user code is dexed at runtime. |
| `./gradlew test` | JVM tests across all modules (~2,100 tests). |
| `./gradlew :ide-android:assembleMinified` | R8 experiment only. **Never ship** — not runtime-validated against the toolchain. |

The `mcp` workflow builds the on-device MCP server (the agent's tools over stdio) and runs its tests.

**This fork is Android-only on purpose.** The desktop launcher (`:ide-desktop`), the iOS shell
(`:ide-ios`), the headless CLI (`:build-cli` + `action.yml`), the desktop packaging workflow and the
upstream Play-listing metadata (`fastlane/`) were removed — nothing on a phone runs them. Upstream
changes to those paths are dropped at merge; see `docs/upstream-sync.md`.

## Ground rules

1. **Stay mergeable.** Do not rename internal packages (`dev.ide.*`), Gradle modules, the `.caproj`
   format, the plugin SPI or the `codeassist` color-scheme id. They look like leftovers; they are the
   reason upstream can still be merged. See `REMIX.md`.
2. **Identity strings.** User-visible text says "CodePrimer". Resource *keys* keep their upstream names
   (`help_improve_codeassist`) — renaming a key breaks every locale for no user-visible gain.
3. **No secrets.** Never commit `keystore.properties`, a real keystore, real AdMob ids, or a Supabase
   key with more than the publishable role. `.gitignore` covers the first two; the rest is on review.
4. **Tests.** A bug fix comes with a test that fails before it. New behaviour on the analysis/build
   path needs one too — that code has no visual tell when it breaks.
5. **AI-assisted commits.** Fine, and increasingly normal here — but say so in the message
   (`Co-authored-by: ...`) and make sure a human has read anything touching `PluginSandbox`, agent
   tools, signing/keystore code, the build daemon, or file operations. Those are the parts that can
   damage a device or a user's project.
6. **Commit messages.** Imperative, scoped, in the existing style: `fix(editor): ...`,
   `feat(preview): ...`, `perf(ndk-plugin): ...`.

## Releasing (maintainer)

1. Bump `versionCode` and `versionName` in `app/ide-android/build.gradle.kts`. The name is
   `<upstream base>-cp.<n>`, e.g. `3.34.0-cp.1` — never a bare upstream version, so a bug report
   identifies the build unambiguously.
2. If upstream was merged, update the `Base:` line in `REMIX.md`.
3. Create the release. The tag push is what triggers the build:

   ```sh
   gh release create v3.34.0-cp.1 --target main --title "CodePrimer 3.34.0-cp.1" --notes-file notes.md
   ```

   The **Debug APK** workflow then builds `:ide-android:assembleProfile` on the tagged commit and
   attaches `CodePrimer-<tag>-profile.apk`.
4. Check the run summary for the APK's `sha256` and paste it into the release notes so the download can
   be verified.
5. If the workflow did not fire on the tag, run it manually against the tag:
   `gh workflow run apk.yml --ref v3.34.0-cp.1`.
