# Ignite3D

Ignite3D is a 3D game engine and editor for Windows, with Android and web builds.

## Download

Get the latest **Ignite3D-Setup** from [Releases](../../releases/latest) and run it.

Ignite3D checks this repository for updates. When a new version is out, it offers to install it from **Help → Check for Updates**.

## Publishing an update

1. Create a release whose tag is the new version, like `v2.5.0`.
2. Attach `Ignite3D-Update-<version>.exe`, plus `Ignite3D-Setup-<version>.exe` for new installs.
3. Put the update file's SHA-256 in the release notes as `sha256: <hash>`. Ignite3D refuses an update whose hash doesn't match.
