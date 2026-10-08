# Ignite3D

Ignite3D is a 3D game engine and editor for Windows. It builds games for Windows, Steam, the Microsoft Store, Xbox (in Edge), Android and the web.

## Download

Get the latest **Ignite3D-Setup** from [Releases](../../releases/latest) and run it.

Ignite3D checks this repository for updates. When a new version is out, it offers to install it from **Help → Check for Updates**.

## Claude and other AI apps

Ignite3D links itself to the Claude desktop app automatically. Claude can then build scenes, write scripts, make and rig models, play-test, and make Windows, Steam, Microsoft Store and Xbox builds. Ignite3D shows each action and lets you undo it.

The bridge works like Unity's: the editor defines its commands, and the `ignite3d` command line drives the running editor.

```
ignite3d status                       # is Ignite3D running and ready?
ignite3d command                      # list the editor's commands
ignite3d command create_objects --objects '[{"name":"Box","components":[{"type":"MeshRenderer"}]}]'
ignite3d eval "return engine.gameObjects.length"
ignite3d screenshot --output shot.jpg
ignite3d play | stop
ignite3d mcp configure claude-code    # also: claude, cursor, vscode, windsurf (--list, --remove)
```

To add the `ignite3d` command to your terminals, open the **Claude app** window in Ignite3D (the spark button) and click **Add the ignite3d command to terminals**. The old Ignite3D extension (`.mcpb`) is retired. If it's still installed in the Claude app, uninstall it there.

## Online games

**Build → Online** deploys Ignite Online, your own game server, to your Cloudflare account. You need a free Cloudflare account and an API token with *Workers Scripts: Edit*, plus *Cloudflare Realtime: Edit* for TURN relays. It gives your games:

- **Multiplayer** (`Net`): rooms with codes, quick match, peer-to-peer connections with TURN relays, a new host when the host leaves, and the Network Sync component.
- **Cloud saves** (`Cloud`): saves that follow the player between devices and work offline.

## Ads

**Build → Ads** sets up AdMob banner, interstitial and rewarded ads for Android builds. Google's test ads are on until you turn them off. In Play Console, declare that the app contains ads and fill in Data safety as Google's AdMob guide describes.

## Steam

**Build → Build for Steam** makes a Windows game with Steamworks (achievements, stats, Steam Cloud saves, the overlay) and a **Steam upload** folder next to it with the SteamCMD build script and **Upload to Steam.bat**. You need a Steamworks account and an App ID for your game. In scripts:

```js
Steam.unlock('ACH_FIRST_WIN');
Steam.addStat('kills', 1);
await Steam.cloudSave('slot1', { level: 3 });
const save = await Steam.cloudLoad('slot1', null);
Steam.overlay('achievements');
```

## How updates work

Ignite3D reads `updates/latest.json` in this repository, which names the newest version, its update file and its SHA-256 hash. It downloads that small `Ignite3D-Update-<version>.exe` from `updates/`, checks the hash, installs it and restarts. An update whose hash doesn't match is refused.

To publish an update, add the new update `.exe` to `updates/` and point `latest.json` at it. GitHub releases also work: tag `vX.Y.Z`, attach the update `.exe`, and put `sha256: <hash>` in the notes. Ignite3D uses those only when `latest.json` is missing.

## Releases

Each version also has a [release](../../releases) with the full installer, a zip of the app and the update `.exe`. The Release workflow (`.github/workflows/release.yml`) makes it: push a branch `release/vX.Y.Z` from main with the installer in `release-files/` (split into parts under 100 MB, plus `SHA256SUMS`), and the workflow joins the parts, checks them, creates release `vX.Y.Z` on main with notes from `latest.json`, and deletes the branch.
