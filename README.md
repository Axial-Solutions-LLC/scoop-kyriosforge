# scoop-kyriosforge

[Scoop](https://scoop.sh) bucket for [KyriosForge Desktop Companion](https://axialsolutions.io)
— a self-hosted container fleet management platform.

## Install

```powershell
scoop bucket add kyriosforge https://github.com/MaverickTN/scoop-kyriosforge
scoop install kyriosforge/kyriosforge
```

## How this manifest works

The real installer (`packaging/windows/installer.iss` in the app's own repo) is
an Inno Setup `.exe` that does a normal per-user install under
`%LocalAppData%\Programs\KyriosForge`. This manifest does **not** run that
installer. `"innosetup": true` tells Scoop to extract the files the installer
would have placed under `{app}` directly out of the `.exe` via `innounp`
(a dependency Scoop installs automatically) — the same mechanism Scoop uses
for a portable zip. Nothing is written to the registry and no uninstaller
script runs; `scoop uninstall` just removes the extracted folder and the
`KyriosForge.exe` shim. Confirmed against Scoop's own source
(`lib/decompress.ps1`'s `Expand-InnoArchive`, `lib/depends.ps1`), not assumed.

The app's own per-user settings (hub address, API token) live in
`%AppData%\KyriosForge\credentials.json`, outside the Scoop app directory
entirely, so an upgrade (new version, new extracted folder) never touches
them — no `persist` entry needed.

**Not yet verified on a real Windows machine** — this environment has no
Windows host to run `scoop install` against. Before announcing this bucket
anywhere, do a real install/upgrade/uninstall pass on Windows: confirm
`innounp` actually unpacks this specific Inno Setup build (LZMA2/ultra64,
solid compression) cleanly, the shim launches the app, and the shortcut
shows up in the Start Menu.

## Updating for a new version

There's no automated `checkver`/`autoupdate` here yet — KyriosForge doesn't
publish a plain version-string endpoint for the desktop app (only
version-pinned/`latest/` binary URLs), so there's nothing for Scoop's
autoupdate machinery to regex a version out of. Bump by hand:

1. Confirm the new version is live: `packaging/build-windows-release.sh` +
   `packaging/s3-publish.sh` in the app's own repo publish to
   `https://dl.kyriosforge.io/releases/kyriosforge-forge-app/<version>/kyrios-forge-app-windows-x86_64.exe`,
   with a `.sha256` file alongside it.
2. Edit `bucket/kyriosforge.json`: bump `version`, update
   `architecture.64bit.url` to the new version's path, and set `hash` to
   that version's published sha256 (uppercase/lowercase doesn't matter to
   Scoop, but download the file and verify it yourself rather than trusting
   the posted `.sha256` blindly).
3. Commit and push — no PR/moderator queue, this bucket is self-hosted.
