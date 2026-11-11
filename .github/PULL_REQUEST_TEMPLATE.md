## What this changes

<!-- One or two sentences. Link the issue if there is one. -->

## Why

<!-- The behaviour that was wrong, or the capability that was missing. -->

## How it was verified

<!--
A "builds successfully" line is not verification for this project: the compiler, dexer and signing path
only fail at runtime, on a device, against a real project.
-->

- [ ] `./gradlew test` passes (or: which module's tests I ran)
- [ ] Built and installed `:ide-android:assembleProfile` on a device
- [ ] Exercised the affected path end to end (state which: edit → build → dex → sign → install → run)

## Upstream sync

- [ ] This change is CodePrimer-specific, **or** it is generic and worth sending upstream
- [ ] No internal names renamed (`dev.ide.*`, module names, `.caproj`, plugin SPI, `codeassist` color scheme)
- [ ] No secret, keystore, real ad id or backend key added
- [ ] If this came from AI assistance on a risky path (`PluginSandbox`, agent tools, signing/keystore,
      build daemon, file ops), a human has read it line by line
