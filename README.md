# Ignite3D

Ignite3D is a 3D game engine and editor for Windows, with Android and web builds.

## Download

Get the latest **Ignite3D-Setup** from [Releases](../../releases/latest) and run it.

Ignite3D checks this repository for updates. When a new version is out, it offers to install it from **Help → Check for Updates**.

## Claude

Ignite3D works with the Claude app. You can open [`extension/Ignite3D.mcpb`](extension/Ignite3D.mcpb) with the Claude app to install the Ignite3D extension, or click **Connect** in Ignite3D's Claude panel. Then ask Claude to build in Ignite3D.

## How updates work

Ignite3D reads `updates/latest.json` in this repository, which names the newest version, its update file and its SHA-256 hash. It downloads that small `Ignite3D-Update-<version>.exe` from `updates/`, checks the hash, installs it and restarts. An update whose hash doesn't match is refused.

To publish an update, add the new update `.exe` to `updates/` and point `latest.json` at it. GitHub releases also work: tag `vX.Y.Z`, attach the update `.exe`, and put `sha256: <hash>` in the notes. Ignite3D uses those only when `latest.json` is missing.
