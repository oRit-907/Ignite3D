# Ignite3D

Ignite3D is a 3D game engine and editor for Windows, macOS and Linux. It builds games for Windows, Steam, the Microsoft Store, Xbox (in Edge, or as an app on a console in Dev Mode), Android and the web, and plays them in a real Android emulator inside the editor.

## Download

From [Releases](../../releases/latest):

- **Windows:** `Ignite3D-Setup-<version>.exe`. Run it.
- **Linux:** `Ignite3D-<version>-linux-x64.AppImage`. Make it executable (`chmod +x`) and run it.
- **macOS:** `Ignite3D-<version>-mac-arm64.zip` (Apple silicon) or `-mac-x64.zip` (Intel). Unzip and move Ignite3D.app to Applications. The app isn't notarized yet, so the first time right-click it → Open, or run `xattr -cr /Applications/Ignite3D.app`.

Ignite3D checks this repository for updates on every system. When a new version is out, it offers to install it from **Help → Check for Updates**: Windows installs a small update, Linux replaces the AppImage, macOS replaces the app.

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

## Scripting

**New script** asks which language to use:

- **JavaScript:** `class Player extends Behaviour { update(dt) { … } }`
- **C#:** Unity-style MonoBehaviours. Classes, structs, generics, LINQ, coroutines and async/await, with GameObject, Transform, Rigidbody, Input, Physics, PlayerPrefs and SystemInfo. Public fields show in the Inspector.
- **GDScript:** Godot 4 syntax, and Godot 3 forms. CharacterBody3D with `move_and_slide`, RigidBody3D and Area3D signals, timers, tweens, `await`, input actions, autoloads and `$Child` paths.
- **Visual script:** a Blueprint-style node graph (events, flow, physics, sound, input, math), edited with mouse or touch.

Every language runs wherever the game runs (web, Android, Windows, Xbox, the C++ player), errors point at the line or node, and Claude can write them all. **Window → Audio Mixer** sets up mixer groups. Scripts play sounds with `Audio.play`, `playAt` (3D) and `playMusic` (crossfades).

## C++ player: DirectX 11, Vulkan, NVIDIA and AMD

**Build for Windows → Player: C++** makes a small `.exe` that runs the game without a browser engine. **Graphics** picks what draws it:

- **Auto (DirectX 11)**, **DirectX 11** or **Vulkan:** through [ANGLE](https://chromium.googlesource.com/angle/angle), the layer Chrome uses for WebGL on Windows. Its DLLs (about 13 MB) go next to the `.exe`.
- **OpenGL:** the graphics driver's own OpenGL 3.3, with nothing extra.

If the chosen renderer can't start on a PC, the game falls back (Vulkan → DirectX 11 → OpenGL) and `player.log` says why. **GPU** picks the fast GPU (default) or the power-saving one on laptops with NVIDIA Optimus or AMD switchable graphics. `player.log` lists every GPU and the one in use. Players can override both: `Game.exe --renderer vulkan --gpu power-saving`. Games read the GPU with `Engine.graphics`, `SystemInfo` (C#) or `RenderingServer` (GDScript).

**Build for Android → Player: C++ (experimental)** makes a small native Android app on OpenGL ES 3 with the same engine. Its JavaScript engine has no JIT, so keep the WebView player for store releases for now.

## Xbox services and the Microsoft Store (Microsoft GDK)

**Build for Windows → Player: C++ → Xbox services** gives Windows games the player's Xbox sign-in and gamertag, title-managed stats and leaderboards, rich presence, achievements (ID@Xbox titles) and Xbox cloud saves. Scripts use `Xbox.signIn()`, `Xbox.setStat`, `Xbox.leaderboard`, `Xbox.setPresence`, `Xbox.unlock` and `Xbox.cloudSave` / `Xbox.cloudLoad` in any language. In the editor they run in a test mode, so you can try them before the game is set up. **Xbox cloud saves** also keeps the game's `localStorage` in the player's Xbox cloud.

You need the free [Microsoft GDK](https://github.com/microsoft/GDK/releases) on your PC (`winget install Microsoft.Gaming.GDK`) and your game set up in Partner Center (the Xbox Creators Program is open to everyone; achievements need ID@Xbox). Enter the Title ID, SCID and MSA App ID; the build copies the GDK's runtime from your install and writes `MicrosoftGame.config`. Ignite3D doesn't include any part of the GDK. Under the GDK's license, games that use it are published through the Microsoft Store.

**Package: Microsoft Store (GDK)** makes the `.msixvc` with the GDK's `makepkg`: a test package that installs on your PC with one button, or a package encrypted for upload in Partner Center.

The C++ player reads controllers through **Microsoft GameInput** on Windows (Xbox, PlayStation and more), with rumble and the Xbox controller's trigger rumble: `Input.vibrate(strong, weak, ms, leftTrigger, rightTrigger)`. PCs without GameInput use the standard controller driver.

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

## Android emulator

In the desktop app, **Preview → Android** runs your game's real APK on Android, in Google's Android Emulator, with its screen right in the editor: click to touch, type on the keyboard, and use Back, Home, rotate and the device menu under it. The game's console messages show in Ignite3D's Console. Set it up once in **Preview → Android Emulator…**:

- **Android OS:** use your own (a system image `.zip` like the ones Android Studio downloads, or its unpacked folder, which is used where it is and takes no extra space), use the Android SDK already on your computer (Android Studio's), or download one from Google. The AOSP images are the smallest (about 0.7–0.9 GB). Images are unpacked sparse, so empty space inside them takes no disk space.
- **Emulator program:** from your Android SDK, or downloaded from Google once (about 350–490 MB). Google's license is shown before any download.
- **Needs:** hardware virtualization (Windows: turn on *Windows Hypervisor Platform* in Windows features; Linux: KVM), and about 7.5 GB free where the emulator keeps Android's storage the first time it starts (it only uses what Android writes). **Storage** in the same window shows what's used and removes it; **Change folder…** puts it on another drive.

## Xbox

**Build → Build for Xbox** makes either a web game for Microsoft Edge on Xbox or the **Windows & Xbox app**: a Visual Studio project (UWP, C#) that runs the game in WebView2. The app is one package for PC and Xbox, and the one to submit to the Microsoft Store.

With your Xbox in Dev Mode and Visual Studio 2022 or newer with the *Universal Windows Platform development* workload and a Windows SDK on your PC, **Build and run on Xbox** builds the package, installs it on the console and starts it:

1. On the Xbox, open Dev Home. Under Remote Access, turn on Device Portal, set a user name and password, and note the IP address.
2. In Ignite3D, choose Windows & Xbox app, enter the IP, user name and password, and press **Connect**.
3. Press **Build and run on Xbox**. Each build installs over the last one and keeps the game's saves.

The password stays in memory until Ignite3D closes. Ignite3D remembers the console's security certificate and sends nothing to a console whose certificate has changed until you press **Trust this console**. Once, in Dev Home, set the game's App type to Game for the console's full memory and graphics.

## How updates work

Ignite3D reads `updates/latest.json` in this repository, which names the newest version, its update file and its SHA-256 hash. It downloads that small `Ignite3D-Update-<version>.exe` from `updates/`, checks the hash, installs it and restarts. An update whose hash doesn't match is refused.

For Linux and macOS, `latest.json` also has a `platforms` section (`linux`, `mac`, `mac-arm64`) naming the release's AppImage or app zip, its download address, size and SHA-256. Ignite3D downloads that package, checks the hash and swaps itself.

To publish an update, add the new update `.exe` to `updates/` and point `latest.json` at it. GitHub releases also work: tag `vX.Y.Z`, attach the update `.exe`, and put `sha256: <hash>` in the notes. Ignite3D uses those only when `latest.json` is missing.

## Releases

Each version also has a [release](../../releases) with the full installer, a zip of the app and the update `.exe`. The Release workflow (`.github/workflows/release.yml`) makes it: push a branch `release/vX.Y.Z` from main with the installer in `release-files/` (split into parts under 100 MB, plus `SHA256SUMS`), and the workflow joins the parts, checks them, creates release `vX.Y.Z` on main with notes from `latest.json`, and deletes the branch.
