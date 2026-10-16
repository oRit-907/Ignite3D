# Ignite3D

Ignite3D is a 3D and 2D game engine and editor for Windows, macOS and Linux. It builds games for Windows, Steam, the Microsoft Store, Xbox (in Edge, or as an app on a console in Dev Mode), Android and the web, and plays them in a real Android emulator inside the editor.

## Download

From [Releases](../../releases/latest):

- **Windows:** `Ignite3D-Setup-<version>.exe`. Run it.
- **Linux:** `Ignite3D-<version>-linux-x64.AppImage`. Make it executable (`chmod +x`) and run it.
- **macOS:** `Ignite3D-<version>-mac-arm64.zip` (Apple silicon) or `-mac-x64.zip` (Intel). Unzip and move Ignite3D.app to Applications. The app isn't notarized yet, so the first time right-click it → Open, or run `xattr -cr /Applications/Ignite3D.app`.

Ignite3D checks this repository for updates on every system. When a new version is out, it offers to install it from **Help → Check for Updates**: Windows installs a small update, Linux replaces the AppImage, macOS replaces the app.

## Projects and scenes

Ignite3D saves a project as a folder, laid out like a Unity project, so it works with git and other tools:

```
MyGame/
  MyGame.ignite                  open this (or the folder)
  ProjectSettings/               build settings, scenes in the build, audio mixer
  Assets/Scenes/Main.scene       a scene, one object per line (+ Main.scene.meta)
  Assets/Scripts/Player.cs       scripts as source files
  Assets/Textures/Ground.png     images, sounds and models as real files
```

Every asset has a `.meta` file that keeps its id, so objects keep their references when assets are renamed or moved. Files you copy into `Assets/` are imported when the project opens. The **Assets** panel works like Unity's Project window: folders, drag and drop, search. Single-file `.ignite.json` projects still open, save and export (**Build → Export Project**).

A project can have many **scenes** (levels, menus): switch them in the Hierarchy, make one with **File → New Scene**, and set the game's scenes and their order in **Build → Scenes in Build**. Scripts load scenes while playing (`SceneManager.LoadScene("Level2")` and `DontDestroyOnLoad` in C#, `this.loadScene('Level2')` in JavaScript, `change_scene_to_file()` in GDScript, or the **Load Scene** node).

**File → New 2D Project** makes a 2D game: an orthographic camera, Sprite Renderers (sprite sheets, tiling, pixel-art filtering), 2D physics and the 2D Scene view. **Pixel Hop** (File → Load Sample) is a 2D platformer to start from.

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
- **C#:** Unity-style MonoBehaviours. Classes, structs, generics, LINQ, coroutines and async/await, with GameObject, Transform, Rigidbody, Input, Physics, SceneManager, PlayerPrefs and SystemInfo (and Rigidbody2D, Physics2D and SpriteRenderer in 2D projects). Public fields show in the Inspector.
- **GDScript:** Godot 4 syntax, and Godot 3 forms. CharacterBody3D with `move_and_slide`, RigidBody3D and Area3D signals, timers, tweens, `await`, input actions, autoloads and `$Child` paths.
- **Visual script:** a Blueprint-style node graph (events, flow, physics, sound, input, math), edited with mouse or touch.

Every language runs wherever the game runs (web, Android, Windows, Xbox, the C++ player), errors point at the line or node, and Claude can write them all.

**Controllers:** scripts read up to four controllers (`Input.getButton('A', 1)`, `Input.gamepads`, C#'s `Input.GetJoystickNames()`, GDScript's device argument) and hear when one connects or disconnects. HUD buttons work with a controller: the D-pad or left stick moves a focus ring, A presses, B goes back (`UI.focus`, `UI.onBack`; GDScript's `grab_focus()` and focus signals), and Web Game pages can opt in with `data-ig-focus`.

**Engine UI.** Scene settings → UI: **Engine** draws the game's menus and HUD with the engine itself, the same in the editor, the web export, the C++ players and on Xbox: panels, text, images, buttons, bars, sliders, toggles and scrolling lists (`UI.Panel`, `UI.Label`, `UI.Button`, `UI.Bar`…), row and column layout where text wraps to fit, gradients, rounded corners, borders and shadows, SVG art from markup, controller focus, `UI.Animate` and `UI.zoom` for the interface size. Font assets (`.ttf`, `.otf`, `.woff`) work in every player, and `Engine.customRender` lets a game draw its own 3D view under the UI.

**When the player leaves the game**, scripts hear it (`IG.App.on('suspend' | 'resume' | 'hide' | 'show')`, `onApplicationPause` in JavaScript, `OnApplicationPause` / `OnApplicationFocus` in C#, `NOTIFICATION_APPLICATION_PAUSED` in GDScript), so they can save before the system suspends the game.

**Native plugins** bring C or C++ code into a game. A plugin asset holds a WebAssembly build (for the web, the Android web app and the Web player) and a native build per platform (Windows, Linux, Android, and the Xbox app's C++ player), and games call it the same way everywhere: `const p = await IgnitePlugins.load('name'); p.call('fn', [numbers], [typedArrays])`. The plugin header `ignite_plugin.h` describes the format and how to build both kinds. **Window → Audio Mixer** sets up mixer groups. Scripts play sounds with `Audio.play`, `playAt` (3D) and `playMusic` (crossfades).

## Ignite Engine (beta)

The **Ignite Engine** is Ignite3D's new native engine: the scene, physics, rendering, particles, HUD, input and sound run in C++, drawn with **Direct3D 12** or **Vulkan**, and scripts run on top of it. In a build's settings, **C++ engine → Ignite Engine (beta)** picks it for Windows builds and for the Windows & Xbox app (Build for Xbox → C++ player, then the same C++ engine setting; it uses `ignite_engine_uwp.dll`), and **Edit → Play in Ignite Engine** runs the open project in it on this PC.

It's a beta: it plays games like the samples (Ember Run, Pixel Hop) the same as the classic C++ player, but scene changes, raycasts, the engine-drawn UI, Web Game objects, ads, Steam and Xbox services still need the classic player, and it hasn't been tested on a console yet. Android builds on it aren't in this version. The classic C++ player stays the default.

## Native Player (beta)

The **Native Player** is Ignite3D's new C++ core: one player for Windows, the Microsoft Store (GDK) and Android that runs the game from one package, `game.igpk`, drawn with OpenGL ES through ANGLE on Windows and OpenGL ES 3 on Android, with no browser engine. It plays games like the samples (Ember Run, Pixel Hop) frame for frame the same as the classic players. Pick it with **Player: Native Player (beta)**:

- **Build for Windows → Game folder:** `<Game>.exe`, `game.igpk` and the ANGLE DLLs. **Graphics** picks DirectX 11, Vulkan or WARP (DirectX 11's software renderer).
- **Build for Windows → Package: Microsoft Store (GDK):** the same folder on the GDK build of the player, with `MicrosoftGame.config` and the GDK runtime from your install, then the `.msixvc` from the GDK's `makepkg`. With **Xbox services** on it also has the player's Xbox sign-in, stats and leaderboards, presence and Xbox cloud saves, with the same `Xbox.*` script calls as the C++ player.
- **Build for Android:** an APK (Android 10 and newer, 64-bit). Not in 2.28.1 yet: its Android template comes in a later update, and until then Android builds use the WebView or C++ player.
- **Build → Export Native Player Package (.igpk):** the game package on its own, also from the browser editor.

It's a beta: there's no Linux version, no Android App Bundle (AAB) for Google Play and no Steam build yet, and Web Game objects need the classic players. Saves live in `%LOCALAPPDATA%\Ignite3D\<exe name>` on Windows (an installed Store package uses its package name instead), so switching a game from the classic player starts with no saves. The classic players stay the default.

## C++ player: DirectX 11, Vulkan, NVIDIA and AMD

**Build for Windows → Player: C++** makes a small `.exe` that runs the game without a browser engine. **Graphics** picks what draws it:

- **Auto (DirectX 11)**, **DirectX 11** or **Vulkan:** through [ANGLE](https://chromium.googlesource.com/angle/angle), the layer Chrome uses for WebGL on Windows. Its DLLs (about 13 MB) go next to the `.exe`.
- **DirectX 12:** ANGLE's DirectX 11 renderer on a DirectX 12 device (Windows 10 and 11); falls back to DirectX 11.
- **OpenGL:** the graphics driver's own OpenGL 3.3, with nothing extra.

If the chosen renderer can't start on a PC, the game falls back (Vulkan → DirectX 11 → OpenGL) and `player.log` says why. **GPU** picks the fast GPU (default) or the power-saving one on laptops with NVIDIA Optimus or AMD switchable graphics. `player.log` lists every GPU and the one in use. Players can override both: `Game.exe --renderer vulkan --gpu power-saving`. Games read the GPU with `Engine.graphics`, `SystemInfo` (C#) or `RenderingServer` (GDScript).

**Build for Android → Player: C++ (experimental)** makes a small native Android app on OpenGL ES 3 with the same engine. Its JavaScript engine has no JIT, so keep the WebView player for store releases for now. It isn't in 2.28.1: its Android template is being rebuilt without the old built-in ads, so Android builds use the WebView player in this version.

## Xbox services and the Microsoft Store (Microsoft GDK)

**Build for Windows → Player: C++ → Xbox services** gives Windows games the player's Xbox sign-in and gamertag, title-managed stats and leaderboards, rich presence, achievements (ID@Xbox titles) and Xbox cloud saves. Scripts use `Xbox.signIn()`, `Xbox.setStat`, `Xbox.leaderboard`, `Xbox.setPresence`, `Xbox.unlock` and `Xbox.cloudSave` / `Xbox.cloudLoad` in any language. In the editor they run in a test mode, so you can try them before the game is set up. **Xbox cloud saves** also keeps the game's `localStorage` in the player's Xbox cloud.

You need the free [Microsoft GDK](https://github.com/microsoft/GDK/releases) on your PC (`winget install Microsoft.Gaming.GDK`) and your game set up in Partner Center (the Xbox Creators Program is open to everyone; achievements need ID@Xbox). Enter the Title ID, SCID and MSA App ID; the build copies the GDK's runtime from your install and writes `MicrosoftGame.config`. Ignite3D doesn't include any part of the GDK. Under the GDK's license, games that use it are published through the Microsoft Store.

**Package: Microsoft Store (GDK)** makes the `.msixvc` with the GDK's `makepkg`: a test package that installs on your PC with one button, or a package encrypted for upload in Partner Center.

The same package works with **Player: Native Player (beta)** (see Native Player above), with or without Xbox services. It needs the GDK from October 2025 (2510) or newer.

The C++ player reads controllers through **Microsoft GameInput** on Windows (Xbox, PlayStation and more), with rumble and the Xbox controller's trigger rumble: `Input.vibrate(strong, weak, ms, leftTrigger, rightTrigger)`. PCs without GameInput use the standard controller driver.

## Online games

**Build → Online** deploys Ignite Online, your own game server, to your Cloudflare account. You need a free Cloudflare account and an API token with *Workers Scripts: Edit*, plus *Cloudflare Realtime: Edit* for TURN relays. It gives your games:

- **Multiplayer** (`Net`): rooms with codes, quick match, peer-to-peer connections with TURN relays, a new host when the host leaves, and the Network Sync component.
- **Cloud saves** (`Cloud`): saves that follow the player between devices and work offline.

## Ads

Ignite3D doesn't ship an ad network. Scripts keep the same ad calls (`Ads.showRewarded`, `Ads.showInterstitial`, `Ads.showBanner` and the rest, in every language), and a game connects its own ad network with `Ads.setProvider({ name, show(format, ad), load, isReady, showBanner, hideBanner, ... })`. Without a provider every ad call answers "no ad": rewarded and interstitial ads resolve `false`, banners don't show, and the first call of a run logs one warning. Builds contain no ad SDK and no advertising ID.

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

**Build → Build for Xbox** makes either a web game for Microsoft Edge on Xbox or the **Windows & Xbox app**: a Visual Studio project (UWP, C#) that runs the game. The app is one package for PC and Xbox. It can go to the Microsoft Store for Windows PCs; the Store on Xbox no longer takes new UWP games, so games reach Xbox players through [ID@Xbox](https://www.xbox.com/games/id) and the Microsoft GDK. Its **Player** is either the **Web player** (WebView2, Microsoft Edge's engine) or the **C++ player** (Ignite3D's own engine on DirectX 11, with no browser engine). Games can sign the player in with the console's Microsoft account (`IgniteHost.microsoftToken`) and sell Microsoft Store add-ons (`IgniteHost.store`) in either player.

With your Xbox in Dev Mode and Visual Studio 2022 or newer with the *Universal Windows Platform development* workload and a Windows SDK on your PC, **Build and run on Xbox** builds the package, installs it on the console and starts it:

1. On the Xbox, open Dev Home. Under Remote Access, turn on Device Portal, set a user name and password, and note the IP address.
2. In Ignite3D, choose Windows & Xbox app, enter the IP, user name and password, and press **Connect**.
3. Press **Build and run on Xbox**. Each build installs over the last one and keeps the game's saves.

The password stays in memory until Ignite3D closes. Ignite3D remembers the console's security certificate and sends nothing to a console whose certificate has changed until you press **Trust this console**. It keeps a list of consoles to switch between, and has **Launch**, **Uninstall from console** and **Show app log** buttons.

**App or Game.** An Xbox runs a UWP app as an *App* (about 1 GB of memory and part of the graphics chip) unless it's a *Game* (about 5 GB and the whole chip). After starting the game, Ignite3D reads which one the console used and tells you. To run games as Games, open Device Portal in a browser, go to **Settings → Preference Settings** and check **Treat UWP apps as games by default**.

The app writes `LocalState\app.log` on the console (the version, memory limit, App or Game, errors and crashes), and recovers when the Web player's browser engine stops. Before the console suspends the game (Home, the Guide, another app), both players let the game's scripts save first.

**Performance on Xbox.** The app shows a loading screen with a progress bar until the game's first frame, and tells games which console they're on (`IgniteHost.device`: Xbox One to Series X, with a low / mid / high tier). **Scene settings → Quality** has presets (Low, Medium, High, Ultra) and *Auto*, which picks one by console, plus render scale, dynamic resolution (lowers the 3D resolution to hold the frame rate), MSAA, shadows and texture filtering; **Preview → Xbox** can simulate each console. Builds for the Windows & Xbox app and the Windows C++ player compress material textures to BC1/BC3 (**Texture compression** in the build settings), which takes about a quarter to an eighth of the memory; sprites, pixel art, HUD images and normal maps stay as they are. The C++ player also starts faster: its engine scripts ship precompiled, and compiled shaders are kept between runs.

**Diagnostics** (Build for Xbox, once connected): **Get logs** downloads the game's logs from the console; **Profile on console** and **Benchmark** run the C++ player with the profiler or a frame-time benchmark and show the results; **Web DevTools** turns on Edge's remote DevTools for the Web player and shows how to connect. On Xbox and in Preview → Xbox, if the controller disconnects the game pauses with "Reconnect your controller" (a setting in Build for Xbox; scripts can turn it off). Games can read their memory limit (`IgniteHost.memoryLimit`). TVs can crop the picture's edges, so on Xbox the game's UI keeps inside a title-safe margin (5% by default, in Build for Xbox; `Screen.safeArea` in C#), and **Preview → Xbox** shows it in the editor. Xbox services (achievements, stats, leaderboards, Xbox sign-in) need the Microsoft GDK, so in the Windows & Xbox app they report that they're unavailable.

## How updates work

Ignite3D reads `updates/latest.json` in this repository, which names the newest version, its update file and its SHA-256 hash. It downloads that small `Ignite3D-Update-<version>.exe` from `updates/`, checks the hash, installs it and restarts. An update whose hash doesn't match is refused.

For Linux and macOS, `latest.json` also has a `platforms` section (`linux`, `mac`, `mac-arm64`) naming the release's AppImage or app zip, its download address, size and SHA-256. Ignite3D downloads that package, checks the hash and swaps itself.

To publish an update, add the new update `.exe` to `updates/` and point `latest.json` at it. GitHub releases also work: tag `vX.Y.Z`, attach the update `.exe`, and put `sha256: <hash>` in the notes. Ignite3D uses those only when `latest.json` is missing.

## Releases

Each version also has a [release](../../releases) with the full installer, a zip of the app and the update `.exe`. The Release workflow (`.github/workflows/release.yml`) makes it: push a branch `release/vX.Y.Z` from main with the installer in `release-files/` (split into parts under 100 MB, plus `SHA256SUMS`), and the workflow joins the parts, checks them, creates release `vX.Y.Z` on main with notes from `latest.json`, and deletes the branch.
