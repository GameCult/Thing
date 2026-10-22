# Thing

This repository is Thing (the Eve display, control and sensor edge for GameCult
apps). Its present tree is Thing 1, kept running for live consumers until each
ports; Thing 2.0 is built here (`docs/thing2-target.md`). The Thing 1 renderer
repos are archived under their Eve names: EveUnity, EveFlutter, EveElectron,
EveTui, EvePlugins and EveConformance.

## Start Here

Value: services publish a surface once, as a typed retained document
(`gamecult.eve.surface.v1`), and every runtime lowers it locally. Providers keep
authority over state and commands; Eve owns projection, input capture and
timestamped sensor publication.

Problem: without a shared surface contract, each service hand-builds its own
dashboard and each renderer re-implements layout, binding and commands.

Live (Thing 1):

- `packages/org.gamecult.eve.surface`: renderer-neutral C# surface contract and
  composition builders; tests in `tests/GameCult.Eve.Surface.Serialization.Tests`.
- `packages/eve-contracts`: TypeScript types and validators generated from
  `schemas/` (`npm run generate`, `npm test`).
- `packages/eve-browser-lowering`: browser DOM/CSS lowering of surface documents
  (`npm test`).
- `web/`: browser reference oracle (`web/index.html`, `web/eve-dsl.js`; tests
  with `node --test web/eve-dsl.test.mjs`).
- `docs/surface-contract-v1.md`: the contract. `docs/repo-strategy.md`: repo
  ownership. `docs/parity-testing-harness.md` with
  `scripts/run-parity-harness.ps1`: parity runs.
- `android/` and `Sources/` (iOS Theos app, `make package`): native proof
  clients; build steps under Build Shape below.

Planned (Thing 2.0, in progress): `crates/thing-contract` is the first Rust
piece. The layout solver, text, accessibility tree, closed render-command list
and thin backends described in `docs/thing2-target.md` and `docs/thing2-map.md`
are not built yet; `docs/thing-campaign.md` holds the rename plan.

CultMesh-based surface web, portable control layer, and timestamped sensor
edge.

Eve is the display/control/sensor edge for GameCult apps. Every participant in
a Verse should be able to publish a structured control surface, live data tree,
media panel, story scene, or operator dashboard through CultMesh. Eve clients
render those surfaces, return operator intent, and publish local sensors with
timestamps so Mimir, VoidBot, Fensalir, Sai, Odin, StreamPixels, and the rest of
the mesh can share one inspectable field of state.

The promise is the Eve MultiVerse: chosen CultCache backing stores become Verse
consensus shards; annotated typed documents become reactive state; Eve surface
documents bind to those document fields; users touch native-feeling controls on
browser, iOS, Android, Fensalir Direct2D, Unity, overlays, and future clients;
providers still own accepted truth. Eve makes distributed state visible,
touchable, and answerable without stealing authority from the system that owns
the consequence.

In the GameCult service architecture, Eve is the shared interactive language:
services publish meaningful GUI/TUI surfaces once as Eve DSL or
`gamecult.eve.surface.v1`, and each runtime lowers that surface locally.
Durable service state belongs in CultCache `.cc`; local Verse visibility belongs
in CultMesh; discovery belongs in Odin.

The browser implementation is the ground-truth renderer and behavior reference.
Native clients on iOS, Android, and other surfaces should match the browser's
CultMesh document semantics as closely as possible, using platform-native UI
and media paths where that makes the surface faster, more direct, or more
capable.

This repo is part of the wider public argument already visible on
`gamecult.org`: the [integrated dossier](https://gamecult.org/dossier) names
CultMesh as typed distributed state; the
[Week 07 damage report](https://gamecult.org/Blog/daily-damage-report/week-07)
records Eve, Fensalir, CultLib, Sai, and Odin turning operator surfaces into
first-class state; [The Free Mouth And The Native Body](https://gamecult.org/Blog/the-free-mouth-and-the-native-body)
describes the Persona/native-body split Eve must serve; and
[The Sleeping Colossus Learns To Refuse The Throne](https://gamecult.org/Blog/the-sleeping-colossus-refuses-the-throne)
is the praxis warning: means and ends must match. Eve's means are local
projection, consentful command boundaries, inspectable stale state, and clear
provider authority.

## Objective

Build one deployable Eve runtime family:

- Browser: canonical CultMesh UI compositor, test oracle, and reference
  behavior.
- iOS: native full-screen renderer, multitouch controller, camera/mic/motion
  sensor publisher, and low-latency media display.
- Android: native renderer and sensor publisher with the same CultNet contracts.
- Flutter: likely native client framework for Android/iOS parity once installed;
  it should consume the browser-defined CultMesh surface model rather than
  replacing the browser as ground truth.
- Fensalir Direct2D: close-to-metal desktop/runtime client surface for the same
  CultMesh UI documents, lowered through Fensalir's existing
  DirectWrite/Direct2D overlay machinery.
- Shared API: apps publish control surfaces and structured data; Eve publishes
  commands, pointer/touch input, and timestamped sensor packets.

## Philosophy

Eve is not remote DOM streaming in a new hat. Remote pixels are tolerated only
where they are the honest edge of an existing system. The durable shape is a
retained surface document backed by typed CultCache state and carried through a
Verse.

The user story should be this simple:

```csharp
using var node = await CultMesh.StartNodeAsync("mimir.ccmp");

var surface = node.Documents
    .Document<MimirProgramSurfaceConfigDocument>("mimir.program.default");

element.Bind(
    surface.Field(x => x.Layers[1].Opacity),
    access: CultAccess.ReadWrite);
```

That exact ergonomic API is still a target, not a completed substrate feature.
The existing CultMesh/CultNet database layer already exposes typed document
watch streams such as `Watch<T>()`, `WatchRecord<T>()`, `WatchByName<T>()`, and
`WatchByIndex<T>()`; the missing Eve-facing cut is field-level binding over
annotated POCOs, authority labels, and native renderer adapters that can expose
staleness, denial, prediction, and reconciliation without making every provider
write dashboard plumbing by hand.

The invariant is blunt: CultCache owns typed document identity and local
persistence; CultMesh owns Verse visibility, shard authority, leases,
subscription fanout, and reconciliation; providers own domain truth and side
effects; Eve owns projection, input capture, timestamped sensor publication,
and renderer parity. If a pretty panel cannot say which fields are stale, denied,
predicted, or authoritative, it is not done. It is flirting with the operator.

The renderer-neutral C# contract lives at
`packages/org.gamecult.eve.surface`. Provider daemons and lowerers consume that
Eve-owned package; renderer repositories do not own copies of the surface
schema or composition builders.

## Current Mechanism

The current checked-in client is the iOS Theos app, still named `EveCanvas` at
the bundle level until the wider Eve runtime split exists.

The Android proof under `android/` is a small native Kotlin client built with
the installed Android SDK and CultLib's `cultmesh-kotlin` package, without
Gradle or Flutter. It exists to put Eve on Periwinkle immediately: it consumes
typed `mimir.eve_dashboard_state.v1` CultMesh documents, sends
`mimir.eve_dashboard_command.v1` command documents, and publishes timestamped
touch/motion observations as `mimir.eve_sensor_observation.v1` plus camera/mic
payloads as `mimir.eve_media_observation.v1`.

- `EVEAppDelegate` creates one fullscreen `UIWindow`.
- `EVEViewController` installs:
  - `EVEGLView`, a `CAEAGLLayer` backed OpenGL ES render target.
  - a full-screen `UIImageView` that displays CEF relay frames.
  - `EVEFrameStreamClient`, a native WebSocket client for frame/input transport.
  - a native UIKit `UILabel` overlay for crisp Retina status text.
  - `CADisplayLink` for frame ticking.
  - `CMMotionManager` for accelerometer and gyro telemetry.
- `EVEFrameStreamClient` receives binary JPEG frames from the Starfire CEF relay
  and sends touch events back as JSON viewport coordinates.
- `EVESensorUplinkClient` opens separate WebSocket uplinks for camera and
  microphone observations so sensor traffic does not block display/control
  traffic.
- `EVEDashboardClient` opens the native Mimir dashboard socket and receives
  scene/control state snapshots.
- `EVEViewController` captures camera frames with AVFoundation and microphone
  blocks with AVAudioEngine, then sends binary CultMesh `eve-camera` and
  `eve-mic` media observations to Mimir. It also renders the dashboard natively with UIKit: a scene graph,
  draggable source panels, visibility/reset controls, and multitouch
  pan/pinch/rotate transform commands.

## Invariants

- Browser layout is the reference behavior. Native clients do not improvise new
  semantics when a CultMesh surface already defines them.
- Native clients do not embed a browser as their only answer. They render the
  same surface documents with platform-native controls when that is the better
  machine.
- Fensalir's Direct2D surface is a client renderer, not a second source of UI
  truth. It consumes provider-owned CultMesh surface state and returns commands
  through the same Eve/CultNet path.
- Reactive bindings name typed document fields and authority. Local callbacks,
  DOM handlers, UIKit actions, and Direct2D hit tests are renderer actuators,
  not portable state owners.
- CultNet carries typed surface state, commands, and timestamped sensor packets.
  Edge JSON is tolerated only as an interoperability envelope while the typed
  CultMesh document shape is being proven.
- UIKit is the current iOS streamed-frame owner; OpenGL ES is fallback/local
  render.
- The status bar stays hidden.
- Eve clients own local sensor reads and timestamps. Mimir owns synchronization
  and final interpretation after those samples arrive.
- App-specific dashboards are providers. Eve renders them; the provider owns
  accepted state, commands, and side effects.

See `docs/cultmesh-streaming-ui-framework.md` for the target architecture.
See `docs/repo-strategy.md` for the kernel/plugin/runtime/provider ownership
boundary and graduation rules.
See `docs/eve-migration-roadmap.md` for the staged migration from in-repo
incubation to kernel, plugin, runtime, conformance, and provider-owned repos.
See `docs/renderer-parity.md` for current browser/iOS/Android/Fensalir/Flutter
renderer parity.
See `docs/surface-contract-v1.md` for the shared CultUI/CultMesh surface and
command contract.
Nested CultUI surfaces are documented there as `embeddedDocuments` /
`surface.slot`; the parity fixture is
`web/fixtures/cultui-embedded-surface.json`, and the runtime matrix is in
`docs/parity-testing-harness.md`. Renderer runtimes lower or preserve the slot
directly; Rust proves the same `gamecult.eve.surface.v1` contract at the
CultMesh typed document sync layer in CultLib.
See `docs/eve-dsl-reactive-bindings.md` for the first Eve DSL and reactive
CultMesh binding contract.
See `docs/eve-multiverse.md` for the Eve MultiVerse philosophy and missing API
surface.

## Browser Reference

Start the browser reference surface runner:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\start-browser-reference.ps1
```

Open `http://127.0.0.1:8798/`. Odin-owned Hermodr serves the browser reference
and lowers Odin-visible provider advertisements and surfaces. Local fixtures
remain an offline oracle when the live catalog is unavailable. The page can
render:

- `Fensalir Direct2D`, a fixture surface for the planned
  CultMesh-to-`AquariumUiDocument` lowering.
- `Sai VN Surface`, a fixture for visual-novel scenes exported as
  `gamecult.eve.surface.v1`, including optional nested Norn and TeX sidecar
  plugin surfaces placed diegetically inside the scene when those independent
  plugins are available.
- `Huginn .cc`, a read-only `.eve` fixture emitted by CultCache/Huginn for
  `.cc` files: source witness, schema catalog, persisted records, payload byte
  counts, and decode status. Huginn emits Eve DSL; Eve compiles and renders it.
- `Reactive DSL`, a browser-reference fixture that compiles a small `.eve`
  composition language into the same surface contract and binds UI controls to
  live CultMesh-shaped vars, collections, and derived fields.
- `Embedded Surface Slot`, a CultUI fixture proving that a parent surface can
  reserve layout while a child CultMesh document is resolved and rendered as a
  nested surface.

## VoidBot CEF Stream

The CEF relay that produced these frames is not in VoidBot's default branch, so
no relay command is documented here until one exists.

EveCanvas displays binary JPEG frames and returns touch events only when a
stream lowering is configured through `EVE_STREAM_URLS`. The native app no
longer guesses a Starfire relay address.

## Mimir Sensor Uplink

Run the receiver processes from a Mimir checkout (a sibling `..\Mimir` here):

```powershell
dotnet run --project ..\Mimir\src\Mimir.EveSensorReceiver\Mimir.EveSensorReceiver.csproj -- --port 8793 --path /eve/camera --source-id eve-camera --type video-frame
dotnet run --project ..\Mimir\src\Mimir.EveSensorReceiver\Mimir.EveSensorReceiver.csproj -- --port 8794 --path /eve/mic --source-id eve-mic --type audio-block
```

EveCanvas sends binary `mimir.eve_media_observation.v1` camera and microphone
observations only when `EVE_CAMERA_URLS` and `EVE_MIC_URLS` are configured.
Periwinkle sends camera, microphone, motion, and touch observations only when
launched with an explicit `org.gamecult.eve.SENSOR_URI`; the apps no longer
guess receiver URLs.

## Native Mimir Dashboard

Start the dashboard authority on Starfire:

```powershell
dotnet run --project ..\Mimir\src\Mimir.EveDashboard\Mimir.EveDashboard.csproj -- --port 8795
```

EveCanvas connects to the Mimir dashboard WebSocket lowering for native retained
`dashboard-state` snapshots. That lowering renders the daemon-owned CultMesh
dashboard state; it is not provider discovery authority. Live provider discovery
belongs to Odin/CultMesh advertisements. Native iOS dashboard lowerings must be
configured through `EVE_DASHBOARD_URLS`; missing discovery stays visible instead
of falling back to LAN constants. EveCanvas sends compact commands back:

- `select`
- `move`
- `scale`
- `rotate`
- `toggle-visibility`
- `reset-transform`
- `open-provider`

Eve renders and edits dashboard trees natively; each provider owns accepted
state and command handling. Eve is the operator's hand on the scene graph, not a
second compositor and not a remote WebKit runtime.

The broker includes a native VoidBot tab/provider. Eve renders its CTB rail with
avatar images, selected Persona status panel, state tree, and detail pane from the
same VoidBot `swarm-state.json` projection used by the web dashboard.

## Build Shape

### Android / Periwinkle

Build the Android proof APK from Starfire:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\build-android.ps1
```

Install to Periwinkle, then launch it. If Odin/CultMesh has provided dashboard
and sensor endpoints, pass them explicitly; otherwise the app starts with those
network paths disabled instead of guessing LAN addresses.

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" -s bad9dd01 install -r .\artifacts\android\eve-debug.apk
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" -s bad9dd01 shell am start -n org.gamecult.eve/.MainActivity --es org.gamecult.eve.DASHBOARD_URI "ws://<odin-or-dashboard-lowering>/eve/deck/cultmesh" --es org.gamecult.eve.SENSOR_URI "ws://<odin-or-sensor-lowering>/eve/periwinkle"
```

On Xiaomi/MIUI devices, ADB install may require enabling developer setting
`Install via USB` and approving the on-device prompt.

### iOS / EVE

The project is a Theos-style iOS application:

```sh
make package
make install
```

EVE has SSH, jailbreak package management, a Procursus native build toolchain,
Theos at `/var/theos`, and the packaged iPhoneOS SDK linked under
`/var/theos/sdks`.

Expected EVE target:

```text
ssh alias: eve
device: iPad11,6 / J171aAP
iPadOS: 14.4
install path: /Applications/EveCanvas.app
```

## Deployment

Stage from the workstation:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\stage-to-eve.ps1
```

Build and install on EVE:

```sh
cd /var/mobile/Projects/Eve
export THEOS=/var/theos
make package
make install
uicache -p /Applications/EveCanvas.app
```

Launch `EveCanvas` from SpringBoard, or over SSH:

```sh
uiopen --bundleid org.gamecult.evecanvas
```

Capture the running native renderer over SSH:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\capture-eve-screenshot.ps1
```

If OpenGL ES context creation fails on-device, the app stays alive with the
UIKit overlay instead of aborting launch. The render surface can then be fixed
without losing the basic app deployment path.

## Next Cut

- Connect the existing browser reference to a clean-consumer, networked
  CultMesh sample and use it as the visual/behavior oracle for native clients.
- Split the shared CultNet/CultMesh surface contract from the iOS app code and
  keep `gamecult.eve.surface.v1` as the renderer-facing contract.
- Replace JSON/base64 sensor packets with binary framing once camera and mic
  timing are proven through Mimir.
- Replace the dashboard fixture state with live `MimirPresentationControlState`
  and `MimirSceneEditorState` snapshots.
- Expand the Android Kotlin client from dashboard node rendering into the full
  retained `surface.root` renderer, and add the browser client against the same
  provider/sensor API.
