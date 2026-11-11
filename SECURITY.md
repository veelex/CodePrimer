# Security policy

## Supported versions

Only the newest [GitHub release](../../releases) is supported. `main` is a development branch and gets
no backports — see `docs/upstream-sync.md` for why builds can lag upstream.

## Reporting a vulnerability

Use GitHub's private vulnerability reporting:
**https://github.com/veelex/CodePrimer/security/advisories/new**

Please do not open a public issue, and do not test against devices, projects or accounts you do not own.

Targets: acknowledgement within **7 days**, a fix in the next release, and credit in the advisory unless
you ask otherwise.

## Scope — what actually matters in an IDE

CodePrimer is not a normal app: it compiles arbitrary source, runs a Gradle-like build on device, loads
third-party plugins and hosts an AI agent that can write files and run commands. The interesting failure
mode is one that crosses *out* of the user's own project.

**In scope**

- Sandbox escape or code execution outside the user's project — `PluginSandbox`, agent tools, the build
  daemon, plugin classloading.
- Signing and key material — `KeystoreRegistry`, `JksKeyStore`, `KeystoreCrypto`, the signing configs,
  and anything that could leak or misuse a user's keystore.
- Path traversal or symlink escapes in the VFS, zip import, project import/convert, or file operations
  (`FileOpJournal`, `ProjectBackend`, `ProjectsDocumentsProvider`).
- Secrets in the repository or in a shipped APK.
- A dependency vulnerability reachable from untrusted input (user project, imported archive, plugin,
  remote model output).

**Out of scope**

- A user building and running a project they wrote or downloaded.
- Bugs in upstream CodeAssist itself — report those [upstream](https://github.com/tyron12233/CodeAssist/issues).
- The debug keystore (`app/ide-android/src/main/assets/debug.keystore`), which is the public AOSP debug
  key by design.
- Debug or `profile` builds differing from the signed `release` build.
