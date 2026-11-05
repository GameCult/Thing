# Thing 2.0 map

Campaign `thing2` in the eureka mind. Imagination pass `imagination-thing2-contract`,
session `self-2026-10-10c-eureka`, 2026-10-10. Target `thing2:target:r1`
(`docs/thing2-target.md`). Rulings in force and cited by id:
`thing:ruling:thing2-one-solver-taffy`, `thing:ruling:thing2-thing-owns-text`,
`thing:ruling:wire-rename-freeze-fold`, `thing:ruling:operator-compat-rebuild`,
`thing:ruling:operator-dsl-thing-script`, `thing:ruling:operator-member-names-keep`,
`thing:ruling:operator-aetheria-eve-rename`, and the six `thing2:ruling:*` of
2026-10-10 (`code-home`, `thing-script-form`, `advertisement-owner`,
`port-cutover`, `campaign-repo-limit`, `retired-repo-names`).

This page holds body facts, the model page, the consumer contract at the level
consumers need, the cut order at the level of deliverables, and rationale. The
questions it raises are documents in the mind (`thing2:question:*`); it does not
restate their options. Research for the forks:
`docs/research/thing2-contract-forks-prior-art.md`. Evidence it builds on:
`docs/research/thing2-consumer-census.md` (census, cited as C§n, BD1-BD9),
`docs/research/thing2-foundation-prior-art.md`,
`docs/research/thing2-text-input-a11y-prior-art.md`, and `docs/thing-campaign.md`
(rename table T, edges E1-E7, body facts B1-B28).

## Body facts

Probes P1-P16 ran 2026-10-10, P17-P22 2026-10-12, against `origin/<default>` after `git fetch` unless a
local path is named. Scripts are in the session scratchpad
`imagination-thing2-contract/`.

**P1. The Thing 1 contract is 38 schemas, most of them process machinery.**
`p1_schemas.py` over Eve `schemas/` at `a758787`: 38 files. Eleven describe
conformance, hand-off, split-target, release, capture and witness bookkeeping
(`conformance_*`, `*_handoff`, `runtime_*`, `capability_matrix`,
`provider_scenario`); five describe one renderer's output (`unity_*`,
`electron_shell_projection`, `web_layout_probe`, `tui_grid`); seven describe the
plugin system (`plugin*`). Every schema but `command_invocation`,
`command_result`, `entity_soa_view`, `input_*` and `asset_catalog` sets
`additionalProperties: true`.

**P2. Commands travel in four overlapping envelopes, and the result is hand-built
JSON.** `command.v1` (descriptor, open), `command_invocation.v1` (closed),
`command_receipt.v1` (open, `state` enum `accepted|denied|pending|reconciled`)
and `command_result.v1` (closed: exactly `schema, receipt, transientProjection,
pluginPayload, draftDirective`). Ghostlight `crates/ghostlight-dungeon/src/eve.rs`
at `327cbe3`: `validate_invocation` (`:665-700`) re-checks the envelope by string
comparison; `command_result` (`:734-`) takes `state: &str` and builds the receipt
with `json!`; its own log filter names a state `"completed"` that the receipt
enum does not admit (`:746`). Today it passes only `accepted` and `denied`
(tally of call sites). The browser rejects a flat result whole through `ajv`
(`web/package.json` depends on `ajv`; `eve.rs:723-731` comment). Nothing but
convention stands between a provider and a result the viewer refuses.

**P3. One surface kind has two schemas and is stored as an untyped blob.**
`surface.v1` declares `type: const "surface-state"`; `surface_state.v1` exists
only as code: CultLib `packages/cultcache-ts/src/swarm-documents.ts:82-96`
(members `providerId, title, version, updatedAt, surface: object`) and Odin
`crates/odin-core/src/documents.rs` (`EveSurfaceStateRecord { surface: Value }`;
advertisement and interface binding are `{ value: Value }`). The load-bearing
shape is JSON inside CultCache.

**P4. `interface_binding.v1` is not a binding.** weksa
`scripts/weksa-daemon.mjs:321-341` writes routes (`endpoints`, `routes`,
`cultMeshAddress`) and the whole operator surface into it; its document type
parses anything (`schema: { parse: (value) => value }`, `:418`). It is a third
copy of the advertisement's routes and the surface.

**P5. The text DSL lets the renderer write state.** `web/eve-dsl.js:103-155`:
`var`, `collection`, `derive` (`count`, `latest`) live in a renderer-local mesh,
and `applyAction` appends or sets values there on a button press
(`web/fixtures/reactive-composition.eve`: `button "Append Event" append
swarm.events ...`).

**P6. A domain-looking kind is layout plus a server-side command.** Muninn
`crates/sleipnir-daemon/src/main.rs:1374-1415`: `binding.target` is a labelled
target with `x, y, anchorX, anchorY` over a controller image; "listening" is the
daemon's `pending_learn` state. Absolute positioning (Taffy `position:
absolute` with insets), a button and a bound label compose it.

**P7. Login needs a host effect, not a plugin payload.** Ghostlight
`web/src/heimdall-access-adapter.ts:48-66`: a `pluginPayload` of schema
`heimdall.auth_navigation_receipt.v1` makes the client call
`window.location.assign(url)`; on return it reads the URL and resumes
(`:82`, `:134`).

**P8. Two live viewers render surfaces.** Ghostlight's web client vendors
`@gamecult/eve-browser-lowering` and `eve-contracts` from `vendor/eve`
(`web/package.json`). gamecult-ops `scripts/idunn/idunn-deployment-targets.ps1:262-272`
at `9aed01f`: `nightwing-eve-browser-reference`, `Repo = "Eve"`,
`Status = "runtime-enforced"`, `Deploy = $null`. Odin aggregates and renders
nothing.

**P9. CultCache-rs members are serde types.** CultLib
`packages/cultcache-rs/src/lib.rs:5501-5520`: a `DatabaseEntry` whose member is a
struct holding an enum round-trips through the MessagePack store. Whether the TS,
C# and Python CultCache runtimes decode a recursive tagged union written by Rust
with parity was not established in this pass. It is the first thing the
contract cut proves (see the cut order).

**P10. The stale `.voidbot` lines are still on Eve main.** `a758787`:
`.gitignore:28-30` and `scripts/stage-to-eve.ps1:24` (`--exclude=.voidbot`).
`git grep -n '\.voidbot' -- .gitignore scripts` prints exactly those four lines;
`Sources/EVEViewController.mm` mentions `voidbot.swarm` (no dot) and is not
matched.

**P11. The campaign's repo list has room for the early cuts only.** Mind,
`thing2:campaign:self`: repos `GameCult/Eve`, `GameCult/gamecult-ops`.
`eureka-substrate:cut_spec:cut-campaign-repos.r1` (target-revision repo sets, up
to 16) is in force with no report; both its questions are answered
(`eureka-substrate:ruling:campaign-repo-set-home`, `:campaign-repo-removal`).

**P12. Rust CultCache writes a union as serde's external tag; the C# reference
writes `[key, fields]`. They do not read each other.** Probe pass
`imagination-thing2-contract-cut`, 2026-10-10, local scratch
`imagination-thing2-contract-cut/{rs,cs,ts}` against CultLib `6bdc23c9`. One
record type in all three runtimes: `id`, `version`, `root: Node` with `Node`
one of `Box { id, children: [Node] }`, `Text { id, content }`,
`Use { id, composition, slots: [Node] }`, and `outcomes: [Outcome]` with a unit
arm. Rust (`#[derive(DatabaseEntry)]`, serde enums) writes the payload
`94 af..31 07 81 a3"Box" 92 ...`: each arm a one-entry map keyed by the variant
name, fields positional, a unit arm the bare string `"Denied"`. C# (MessagePack
`[Union(n, ...)]`, the reference) writes `94 af..31 07 92 00 92 ...`: each arm
`[key, [fields]]`, the unit arm `[2, []]`. C# reading the Rust store fails:
`MessagePackSerializationException: Failed to deserialize ProbeSurface value.
<- Unexpected msgpack code 129 (fixmap) encountered.` Rust reading the C# store
fails: `failed to decode CultCache entry "probe.surface" ... a value does not
fit the store format`. Each runtime round-trips its own store. The
`cultcache-rs` derive handles structs only
(`cultcache-rs-derive/src/lib.rs:51`); enum members go through `rmp_serde`.
Owner: CultLib (`thing2:follow_up:cultcache-union-parity`).

**P13. TypeScript CultCache has no typed codec.** `cult-cache.ts:679-692`: with
no `formatter`, a payload decodes to plain arrays and maps and `schema.parse`
sees them raw; the interop test hand-writes a positional formatter per type
(`test/cult-cache.test.ts:1263-1310`). In the probe, a hand-written TS codec
for the union read the Rust store, wrote a store Rust read back equal, and
re-encoded byte-identical to Rust's payload; against the C# store the same
codec refused the `[key, fields]` arm (`node is not a tag map`). A TS reader
matches whichever encoding its author hand-codes.

**P14. Nothing live builds from the six retired family repos.** `git grep` at
`origin/HEAD` of Ghostlight `327cbe3`, Odin `8e1de66`, VoidBot `46d891b`,
Heimdall `7904e17`, Muninn `1d0ea0b`, weksa `d16cff5`, Stonks `3f6e91c`, Vili
`8773c15`, Gjallar `6a92976`, StreamPixels `8d40541`, CultLib, Idunn `6667f0f`,
gamecult-site `8169ee5` and Ymir `f8c0e39` for `EveUnity|EveFlutter|EveElectron|
EveTui|EvePlugins|EveConformance` outside `*.md`: no match. gamecult-ops `9aed01f`
matches only `docs/repo-census-2026-09/synthesis.html`; Eve matches a local
smoke script (`scripts/run-aetheria-conformance-consumer-smoke.ps1:35`, a local
path) and a spec; parked Mimir names them in one configuration file. GitHub,
2026-10-10: all six unarchived; active workflows only on EveFlutter and EveTui
(`runtime.yml`) and EveConformance (`conformance.yml`); `GameCult/Thing` and
`GameCult/AetheriaThing` do not exist; AetheriaEve is archived.

**P15. Ghostlight's deploy compares its Eve clone's origin by exact string.**
gamecult-ops `9aed01f` `scripts/deploy-ghostlight-yggdrasil.sh:20`
(`eve_origin=https://github.com/GameCult/Eve.git`) and `:871-876`
(`require_idunn_repo` refuses "a foreign origin" when `git remote get-url
origin` differs), called at `:883`; the binding template carries the same URL
(`idunn/yggdrasil/bindings/ghostlight.toml.in:14`). Editing either URL without
re-pointing Yggdrasil's Idunn-owned clone stops the next Ghostlight deploy;
leaving both on the old name keeps working through GitHub's rename redirect.
The `Repo = "Eve"` targets (`idunn-deployment-targets.ps1:223`, `:236`,
`:262`) are Thing 1 targets (iPad, Android, Nightwing browser reference) paired
with `LocalPath = "$projectsRoot\Eve"`.

**P16. gamecult-site carries a generated Thing 1 surface.**
`scripts/generate-vn-eve-surface.mjs` writes
`GameCult/static/interactive/gamecult-compound/gamecult-compound.eve-gui.json`
(`gamecult.eve.*` ids), referenced by `GameCult/Pitch.md:15`
(`data-eve-gui=`). The census (C) does not list the site as a consumer.

**P17. The contract's integer leaves are signed because TS and Python have no
unsigned leaf; the C# reference has one.** CultLib `b3bd8dd5`:
`src/GameCult.Caching/CultSchemaTypeNames.cs` names any CLR type (so `UInt32`,
`UInt64` and `Byte` slots exist in the reference); `cultcache-ts`
`src/codec.ts` exports only `int32` and `int64` integer codecs and
`cultcache-py` `leaves.py:147-148` only `int32` and `int64`. A `u64` above
2^63 has no TS or Python reading, so versions stay `Int64` and counts `Int32`,
and the non-negative rule is the contract's, not the leaf's. `Rgba` keeps `u8`
(the reference's `Byte`); the TS and Python packages mirror its 0-255 range
from a refusal golden until CultLib gives them a byte leaf.

**P18. A domain newtype's own `Deserialize` holds its rule at every decode
door, and leaves the bytes unchanged.** Probe `imagination-contract-rs-r2/`
(throwaway commit `fbbdaf79` on `649844aa`, Yggdrasil, 2026-10-12): `Seq(i32)`
refusing below 1 and `Version(i64)` refusing below 0, each `Serialize` the inner
integer and `Deserialize` through `new`. A result payload with seq 0 or -7 is
refused by `rmp_serde::from_slice` ("Seq: at least 1") and by
`cultcache_rs::decode_payload` (opaque: "a value does not fit the store
format"); seq 0 through serde's in-memory `IntoDeserializer` is refused the same
way, so the rule holds for any serde format. With `CommandResult.seq: Seq` and
both composition versions `Version` (an `Int32` slot widened to `Int64`),
`goldens_match` still compares byte-identical, `surface.cc` included. The
derive reads each slot with `seq.next_element::<FieldType>()`
(`cultcache-rs-derive/src/lib.rs`, `slot_codec`, at `b3bd8dd5`), so a field's
type is the one owner of its domain on both paths. `DatabaseEntry` requires
`DeserializeOwned` (`cultcache-rs/src/lib.rs:162`), so the public
`Deserialize` of every kind cannot be removed: it is the decode door.

**P19. Dotted provider ids are the Verse's norm; dotted local names are not
needed.** `git grep` over Ghostlight, VoidBot, Odin, Heimdall, Muninn, Stonks
and Eve (operator checkouts, 2026-10-12): provider ids `voidbot.swarm`,
`odin.allseer`, `stonks.market`, `sleipnir.input-mirror.raven`, `heimdall`;
Thing 1 surface ids repeat the provider as a prefix (`voidbot.swarm.surface`,
`ghostlight.play` in `crates/ghostlight-dungeon/src/mesh.rs:37`). No
`gamecult.thing.*` record exists outside the contract's tests, and Thing 1
records are rebuilt, not read (`thing:ruling:operator-compat-rebuild`), so the
Thing 2.0 key grammar strands nothing. Probe evidence of the collision itself:
Soul's `soul-thing2-contract-rs/rust1.log` KEY lines.

**P20. In C#, the constructor MessagePack calls is the one door, and the decode
recursion fits a default thread inside the bound.** Probe
`imagination-contract-cs-ts/` (throwaway commit `e5030d3b` on CultLib
`b3bd8dd5`, Yggdrasil, image dotnet sdk 10.0, MessagePack 3.1.7, Debug and
Release, log `cs2.log`). A `[MessagePackObject]` class with get-only `[Key]`
properties and one constructor taking the keys in slot order, throwing on
`seq < 1`:
- public constructor: `CultDocumentRegistry` accepts it; `result-pending.bin`
  decodes through `CultDocumentMessagePackSerialization` and re-encodes
  identical; `result-seq-zero.bin` is refused by the constructor through
  `DeserializeUntyped`, through `MessagePackSerializerOptions.Standard`, and
  through `System.Text.Json` (which calls the same constructor); no message
  holds the golden's canary.
- private constructor with `[SerializationConstructor]` under
  `[MessagePackObject(AllowPrivate = true)]`: the same results, the registry
  accepts it, reflection finds no public constructor and no public setter, and
  `System.Text.Json` refuses the type (`NotSupportedException`).
- internal constructor without `AllowPrivate`: the registry refuses the type
  ("no constructor MessagePack can call"), and `CultDocumentRegistry.Shared`
  then refuses every type in the assembly, so one bad type breaks the package.
- `outcome-unknown-arm.bin` is refused through `CultDocumentMessagePackSerialization`
  (`CultUnionGuard`: "Union ... has no arm for key 4"), but decodes with a null
  outcome through `MessagePackSerializerOptions.Standard`: the guard lives only
  in CultLib's options, so a package decodes only through them.
- Stack: a Box-shaped recursive union (3 containers per level) reaches the
  256 bound at 84 levels. Decoding recurses per level: depth 10 decodes on a
  64 KiB thread, depth 84 needs 256 KiB in Debug and 128 KiB in Release, and an
  overflow aborts the process (rc 134). .NET's 1 MiB default thread holds the
  deepest admitted payload with at least 4x margin, so C# carries no part of
  `cultlib-gaps:follow_up:decode-stack-safety`.

**P21. In TypeScript, cultcache-ts runs a document's codec and schema on both
doors, and its recursion fits the default stack.** Same probe commit, image
kotlin (Node 24.21.0), log `ts2.log`. cultcache-ts at `b3bd8dd5` (0.14.0) has
`slots`, `union`, `list`, `lazy`, `unit` and the 256 payload bound;
`decodeDocumentPayload` runs the codec then `schema.parse`, and
`encodeDocumentPayload` runs `schema.parse` then the codec
(`src/document.ts`). A `seq` codec delegating to `int32` with a `>= 1` check,
and a `CommandResult` class with a `#brand` private field, a private
constructor, a codec wrapping its slot codec that mints an instance on decode,
and a `schema.parse` that refuses anything without the brand:
- `result-pending.bin` decodes to an instance and re-encodes identical;
  `result-seq-zero.bin` is refused ("seq (slot 1): Seq must be at least 1");
  `outcome-unknown-arm.bin` is refused ("Union Outcome: unknown arm key 4");
  `result-extra-slot.bin` reads as seq 1; no message holds the canary.
- `structuredClone`, spread and `JSON.parse` of an instance lose the brand,
  and encoding any of them, or an object literal with seq 0, is refused; the
  factory with seq 0 throws.
- `tsc` (strict, NodeNext, ESM importing the CommonJS cultcache-ts) passes
  with `@ts-expect-error` on an object literal and a spread typed
  `CommandResult`, on `new CommandResult(...)`, and on assigning `seq`;
  `structuredClone(result)` and `JSON.parse` still type as `CommandResult`, so
  the runtime brand check is the guard there.
- Stack: the same chain reaches the bound at 84 levels. Depth 84 decodes with
  `--stack-size` 200 and the default 984; at 60 to 120 the codec throws (a
  `RangeError`, wrapped by the codec into its own refusal), never aborting.

**P22. Packaging around the two packages.** Eve `6ae2da0`: `.gitignore`
ignores `packages/*` with one `!` pair per package (lines 2-16), so a new
package directory is invisible to git until it gets its pair, and global
`obj/` (line 17) is where the Thing 1 test project keeps its output
(`BaseOutputPath obj\bin\`). `GameCult.Eve.Surface.csproj` reads CultLib from
`CultLibRoot`, defaulting to the sibling checkout `..\..\..\CultLib`.
`npm view @gamecult/cultcache-ts` answers 404: cultcache-ts is on no registry,
and the consumers that use it (Heimdall, StreamPixels) take it as
`file:vendor/CultLib/packages/cultcache-ts` from a submodule. No file outside
`crates/` and `docs/` names `thing-contract` or `GameCult.Thing.Contract` at
`6ae2da0` except `Cargo.lock`.

## The model page

Every persistent kind Thing 2.0 has. Ids follow rename table T: schema
`gamecult.thing.<x>.v<n>`, mesh key `thing:<family>:<id>`. No row keeps a
compatible eve id (`operator-compat-rebuild`).

| Kind | What names it | What happens to it over time | Who decides (one writer) |
| --- | --- | --- | --- |
| Thing Script document (a published surface) | schema `gamecult.thing.surface.v1`; key `thing:surface:<providerId>.<label>`, where `label` is one segment (non-empty, no `.`), so the key splits at its last dot and is injective while provider ids stay dotted (P19); `version` is an `Int64` at least 0 in the record (P17) | Republished whole when its structure changes, `version + 1`; core ignores a version at or below the one it holds. Live values never cause a republish (they arrive through bindings). Retired when the provider deletes the record. A contract revision is a new `v<n>`; stores holding the old one are deleted and rebuilt by their daemon | The publishing provider. Core and viewers only read |
| Composition | schema `gamecult.thing.composition.v1`; key `thing:composition:<publisher>.<name>.v<n>`, `name` one segment (no `.`), `n` an `Int64` at least 0 | Immutable once published: a change is a new `v<n>` under a new key; references pin the version. The standard library (`thing.std.*`) ships compiled into core and is also published by the Thing repo's release | The publisher (a provider, or the Thing repo for `thing.std`). Core resolves and expands; nothing else writes |
| Binding | not a document: a field of a node in a Thing Script document or composition (`Bind`) | Lives and dies with the document that holds it | Declared by the document's writer; resolved only by core |
| Bound state record | the provider's own schema and key (e.g. `voidbot.swarm_control_state.v1`) | The provider's lifecycle | The provider. Core never writes typed state; a change goes through a command |
| Viewer session state (focus, scroll offsets, editing drafts, selection, IME composition, open overlays) | the core instance; node ids within a resolved document | Not persisted and never published; dies with the session. A draft that must outlive the session is state, and state belongs to a provider | Thing core |
| Command declaration | not a document: an entry of the Thing Script document (`CommandDecl`, id local to the document) | With its document | The document's writer |
| Command invocation | schema `gamecult.thing.command_invocation.v1`; key `thing:invocation:<invocationId>` (a UUID minted by core per user action, also the idempotency key) | Written once, never edited; retained by the provider's command boundary until its final result, then the provider's retention | Thing core, on behalf of the viewer; the transport sets the caller identity |
| Command result | schema `gamecult.thing.command_result.v1`; key `thing:result:<invocationId>.<seq>` (`seq` an `Int32` from 1, refused below 1 on construction and decode; the key is injective because `seq` is the last dot segment and never holds a dot; a pending result may be followed by a final one) | Written once per `seq`; the final outcome ends the invocation | The provider that handles the command, through the contract's constructors only; core validates on receipt |
| Asset catalog | schema `gamecult.thing.asset_catalog.v1`; key `thing:asset_catalog:<providerId>.<catalogId>`, `catalogId` one segment (no `.`); `version` an `Int64` at least 0; asset width and height `Int32` at least 0 | Republished whole, `version + 1`; image nodes reference `catalogId` + asset id | The provider |
| Theme | schema `gamecult.thing.theme.v1`; key `thing:theme:<themeId>` | Versioned like a document; documents reference tokens, never a theme | The host application that embeds Thing (the GameCult brand theme is one such document); `thing.std` default theme in core |
| Provider discovery (today `gamecult.eve.provider_advertisement.v1`) | Not a Thing kind (`thing2:ruling:advertisement-owner`): owned by the Verse discovery owner and renamed once there (`thing2:follow_up:discovery-owner-campaign`) | Until its owner lands, the eve advertisement is read through core's discovery port, unchanged | The discovery owner; the provider writes, Odin aggregates |
| Accessibility tree | not persisted: derived by core per frame from the resolved document and layout (AccessKit tree updates) | Rebuilt on every change | Thing core; backends bridge to the platform |
| Render-command list | not persisted and never published: in-process from core to backend | Per frame | Thing core |
| Geometry and render-command goldens | test fixtures in the Thing repo (`tests/goldens/`) | Regenerated only by a reviewed change to core; a backend never writes one | The Thing repo's core tests |

## The consumer contract

Consumers see four things (target, "The consumer contract"). Each is a set of
types in the contract crate and its language packages; nothing in core is
visible to them (`narrow-consumer-contract`).

### 1. Thing Script

**The node set is closed.** A document's node is exactly one of:

| Node | Carries | Covers today's kinds (census C§7) |
| --- | --- | --- |
| `Box` | layout style (Taffy: display flex, grid or block; direction, wrap, gap, padding, margin; size, min, max in px, %, fr, auto, content; position relative or absolute with insets; overflow visible, clip or scroll), paint (background, border, radius as theme tokens or literal colours), accessibility role from a closed set (group, region, list, list item, heading, dialog, toolbar), label, children | `pane`, `card`, `panel`, `dashboard`, `cockpit`, `rail`, `row`, `grid`, `list`, `partition`, `surface`, `form`, `interface`, `binding.target` layout |
| `Text` | content (literal or `Bind`), text style token, wrap, max lines, selectable | `text`, `stat`, label parts of `metric` |
| `Image` | asset reference (catalog + id), fit, alt text | `image.*`, `avatar` |
| `Button` | content children, `on_activate: CommandRef`, enabled (literal or `Bind`) | `control.button`, `command` (Vili) |
| `Toggle` | checked (`Bind`), `on_change: CommandRef` | `control.toggle` |
| `Slider` | min, max, step, value (`Bind`), commit command | `control.slider`, `control.range` |
| `TextInput` | single or multi-line, value kind text or number, placeholder, initial value (`Bind`), submit command | `control.input.text`, `control.input.textarea`, `control.input.number`, `input.number` |
| `Select` | options (literal or a bound collection), value (`Bind`), command | `input.select`, `control.select`, `control.choice-group` |
| `Repeat` | a bound collection and a template subtree instantiated per item, keyed by item key; template bindings may read item members | `list ... bind <collection>` (BD5) |
| `Overlay` | anchored to a node or the viewport, modal or not, dismiss command | `modal`, `control.popup`, tooltips |
| `Plot` | a bound numeric series, closed marks (line, bar, area), axis ranges | `graph`; sparklines a dashboard consumer expects |
| `Embed` | a host slot kind (string the host declares it supports) and slot id; laid out as a leaf of a given size; draws one opaque host command | `embed.norn`, `embed.tex`, `world.scene*`, `field.*`, `canvas.editor`, game viewports |
| `Use` | a composition reference (`publisher.name.v<n>`) and typed arguments | `metric`, `inspector.kv`, `control.tabs`, `heimdall.access_gate`, `vn.stage`, every domain kind |

Every node has an id (unique within its document after expansion), `visible`
(literal or `Bind`) and an optional accessibility label. A domain need is met by
a composition (`compose-not-kind`); a need no composition can meet is an `Embed`
slot drawn by the host, never a new node.

**Compositions are documents, resolved by core.** A composition declares typed
parameters (text, number, bool, `Bind`, `CommandRef`, child slots) and a body of
nodes that reference them. Core expands `Use` by substitution only: there is no
expression language. Compositions never own commands; a composition that needs
an action takes a `CommandRef` parameter, and the command is handled by the
publisher of the document that uses it. `thing.std` provides the common ones
(`metric`, `card`, `pane`, `dashboard`, `rail`, `key-value`, `stat`), so no
consumer language re-implements them.

**Text form.** Whether Thing Script also has a text syntax beside the typed
document, and which is the authority, is `thing2:question:thing-script-form`.
The node set, compositions and rules here hold under every option.

### 2. Binding (BD1-BD9 collapse to one path)

`Bind { source: StateRef, member: MemberPath, format: Option<Format> }`, where
`StateRef` is `{ provider: Option<ProviderId>, schema: SchemaId, key: RecordKey }`
(absent provider = the document's publisher) or `Item` inside a `Repeat`
template. A bound collection is `{ provider, schema, key_prefix }` (the records of
one type) or a list member of one record. `Format` is closed: number (precision,
percent, unit), duration, timestamp, prefix and suffix text.

| Shape | Fate |
| --- | --- |
| BD1 `stateBindings` | Survives as `Bind`; `routeKind`, `routeDescription` and `sourceId` die (core routes through discovery, a document names no transport) |
| BD2 local drafts | Die as bindings: a draft is core's editing state of `TextInput`, `Slider` or `Select` |
| BD3 `writeCommand`, `captureBindings`, `draftDirective` | Fold into commands: a `CommandDecl` lists the input nodes it captures; a completed result says whether captured drafts clear |
| BD4 world-root props | Leave the contract: an `Embed` slot whose host owns the world view |
| BD5 `var`, `collection`, `derive`, `bind` | `bind` and `collection` survive as `Bind` and `Repeat`; renderer-local `var` and `derive` die (P5): derived values are published by the state's owner |
| BD6 whole-document republish | Survives for structure only (`version + 1`); per-tick republish of values dies |
| BD7 `interface_binding` | Dies (P4): routes are discovery's, the surface is the document |
| BD8 embedded documents | `Embed` |
| BD9 reload and poll | Dies: core subscribes through the CultMesh port |

### 3. Commands (one envelope)

- `CommandDecl { id, payload_schema: SchemaId, captures: [NodeId], confirm: Option<Text> }`
  in the Thing Script document. A node may reference only a declared command.
- `CommandInvocation { invocation_id, surface: (key, version), command, payload, captured: [(NodeId, Value)], client, issued_at, delegation }`.
  `payload` is a typed record of `payload_schema`. The caller's identity is set by
  the transport session and has no field a consumer can write.
- `CommandResult { invocation_id, seq, outcome }`, with `outcome` one of
  `Pending { message }`, `Completed { source_version, clear_drafts, effect }`,
  `Denied { reason, node }` and `Failed { reason }`. `effect` is closed:
  `OpenSurface(key)` or `OpenExternal { url, resume: CommandRef }` (P7). The
  receipt is the result: `command.v1`, `command_receipt.v1`,
  `transientProjection` and `pluginPayload` have no successor.
- Handlers in every language receive a decoded invocation and return an
  `Outcome`; the language package writes the result. No package takes a state
  as a string or converts a map into a result. Decoding is the one other door
  (every CultCache kind is deserializable, P18): it holds the same domain as the
  constructors because each field's type owns its rule. A decoded result can
  name any invocation; binding a result to an invocation core issued is core's
  validation (`thing:ruling:thing2-one-solver-taffy`), not the contract's.

### 4. Advertisement

What Thing needs from discovery is only "this provider exists and is reachable
here": its surfaces are its `gamecult.thing.surface` records and its commands are
declared inside them. Discovery leaves Thing (`thing2:ruling:advertisement-owner`);
the contract has no advertisement type, and core reads discovery through a port.

### Rules that must die under their own mutation

Each is a test or negative check in the cut that lands it; deleting the rule's
code fails it.

1. A result decodes only into `CommandResult`; a record with an extra or missing
   field is refused at decode in every language package (golden fixture with an
   extra field).
2. Core turns a result that fails to decode, or one for an unknown
   `invocation_id`, into a visible `Malformed` state for that invocation and a
   log line; no invocation stays pending past its timeout without one.
3. `outcome` is an enum in every package; no API takes a state as a string.
4. A document with an unknown node kind, an undeclared `CommandRef`, or a
   duplicate node id after expansion is refused whole; nothing falls back along
   a kind path.
5. A `Use` that does not resolve renders a visible placeholder with a diagnostic;
   it is never dropped silently.
6. A `Bind` whose record has a different schema, or whose member is missing,
   renders a typed binding error; no value is coerced.
7. Core holds no API that writes typed state; its only outbound write is a
   `CommandInvocation` (negative check over the core crate).
8. No backend crate or package depends on `taffy` or `parley` (negative check on
   the dependency tree): backends receive positioned commands and shaped glyph
   runs.
9. A lower or equal `version` of a document is ignored.
10. Geometry and render-command goldens for the fixture set are byte-identical
    across runs; a backend's parity test consumes the core's list and asserts it
    drew every command it was given.

### Boundaries

- Consumers depend on the contract packages only. Core, backends and the
  standard library are internal.
- Core reaches CultMesh, the clock, fonts and the platform through narrow ports
  (traits), so a parser, resolver, layout or command test runs without a daemon.
- Backends draw and bridge: input events in, render commands and the
  accessibility tree out. IME uses the platform channel the ruling names (hidden
  textarea on the web).

## Thing 1 kinds and their fate

| Thing 1 id (`gamecult.eve.*`) | Fate |
| --- | --- |
| `surface.v1`, `surface_state.v1` (code only) | Collapse into `gamecult.thing.surface.v1` (P3) |
| `command.v1`, `command_invocation.v1`, `command_receipt.v1`, `command_result.v1`, `operation_payload.v1` | Collapse into `command_invocation.v1` and `command_result.v1` (P2) |
| `local_draft.v1`, `interface_binding.v1` | Die (BD2, P4) |
| `asset_catalog.v1` | Survives as `gamecult.thing.asset_catalog.v1` |
| `provider_advertisement.v1` | Per `thing2:question:advertisement-owner` |
| `plugin.v1`, `plugin_advertisement.v1`, `plugin_abi.request.v1`, `plugin_abi.response.v1`, `plugin_abi_fixture.v1`, `plugin_receipt.v1`, `plugin_witness.v1`, `plugin_handoff.v1` | Die: domain plugins become compositions or `Embed` slots |
| `conformance_*`, `capability_matrix.v1`, `provider_scenario.v1`, `provider_handoff.v1`, `runtime_*`, `local_provider_catalog.v1` | Die: parity is geometry goldens; discovery is not a fixture catalog |
| `unity_*`, `electron_shell_projection.v1`, `web_layout_probe.v1`, `tui_grid.v1` | Die: renderer outputs are not contract kinds |
| `entity_soa_view.v1`, `input_bindings.v1`, `input_capability.v1` | Die (no carrier outside the kernel, B25) |
| `mimir.eve_*`, `cultmesh.eve_surface.v0` | Mimir's own ids, renamed by Mimir's port when it is revived (Mimir is archived) |

## The authority map

- **Owner of geometry, text, focus, editing, accessibility, binding resolution
  and result validation:** Thing core (`one-solver`, `core-owns-text`,
  `live-state-core`).
- **Owner of a surface's content and of its commands' outcomes:** the provider
  that publishes the document.
- **Owner of the contract types:** the contract crate; each language package is
  held to the crate's golden wire fixtures and owns nothing of its own.
- **Derived:** the resolved tree, layout, render-command list, accessibility
  tree and goldens. Viewer session state is core-owned and never published.
- **Demoted:** the browser is no longer the layout oracle; renderers are no
  longer owners of kinds, drafts, pointer resolution or result validation; the
  text DSL's local mesh is no longer a writer of state; `surface_state`,
  `interface_binding` and the receipt are no longer documents.
- **Forbidden writers:** a backend computing layout or shaping text; a provider
  writing a result through anything but the package constructor; core writing
  typed state; a document naming a transport; any compatible eve id; a new node
  kind added for one domain.
- **Shared paths:** every viewer (web shell, Ghostlight's client, TUI, future
  hosts) embeds the same core; every consumer language publishes through a
  package held to the same goldens.
- **Deletion line:** Thing 1's tree in the kernel repo, its six family repos and
  every `gamecult.eve.*` reader and writer in the ported consumers. Each port
  deletes its consumer's Thing 1 path in the same cut.

## Cut order (deliverables)

Per-piece contracts, the build graph with sizes, a test per piece and the prior-art
citations for each choice are in `docs/thing2-build-spec.md`; this list is the
summary.

0. **`stale-voidbot-cleanup`** (Eve main, any time): spec
   `thing2:cut_spec:cut-stale-voidbot-cleanup.r1`. Owned here by
   `thing:follow_up:stale-voidbot-cleanup-on-main`.
1. **Contract** (code home per `thing2:question:code-home`): the contract crate
   with the types above, published schemas `schemas/gamecult.thing.*`, golden
   wire fixtures, and the TypeScript and C# packages held to them (the live
   consumers' languages: Rust, JS/TS, C#). The first proof (P9) was run: a
   Rust-written recursive union does not decode in the C# reference, nor the
   reverse (P12), and TS decodes only what a hand-written codec says (P13). The
   contract waited on CultLib's union parity
   (`thing2:follow_up:cultcache-union-parity`), which is on CultLib main by
   `b3bd8dd5` (P20, P21) and unreleased. Three cuts: `contract-rs`, then
   `contract-ts` and `contract-cs`. The published schema of each kind is its
   CultCache catalog entry, identical in the three packages; no JSON Schema
   files are written, because Thing 2.0 has no JSON reader of these kinds (the
   browser reads them through core, not `ajv`). Python and Kotlin packages
   follow as their own deliverables.
   `contract-rs` landed on Eve main (`6ae2da0`) after two Soul passes
   (`thing2:verdict:cut-contract-rs.s1`, `.s2`). The siblings are mapped
   against `6ae2da0` and run side by side, touching only their own package and
   one `.gitignore` insertion each, at separate anchors:
   `thing2:cut_spec:cut-contract-cs.r2` and `thing2:cut_spec:cut-contract-ts.r2`.
   Waiting, one line each:
   - Cross-runtime store check, after both siblings land: the C# package opens
     a store the TS package wrote and the reverse, which proves the catalog
     type names (`GameCult.Thing.Contract.<RustName>`) agree; neither sibling
     can prove it alone.
   - Publication: when `cultlib-gaps:cut_spec:cut-union-parity-release.r3`
     publishes cultcache-ts, the TS package's dependency moves from the sibling
     `file:` path to the published version, and both packages get a release
     line. Until then both read CultLib from the sibling checkout at the Cargo
     rev (P22).
   - CultLib pin bump: when `cultlib-gaps:follow_up:decode-stack-safety` lands,
     move the pin and run `deep_tree` on the default thread (its 16 MiB thread
     goes); the crate docs' hazard note goes with it.
   - Missing-slot golden: re-added only when
     `cultlib-gaps:follow_up:rust-missing-slot-vs-reference` settles the rule.
2. **Core**, in three cuts: (a) resolution, compositions with `thing.std`,
   binding through the CultMesh port, Taffy layout, render commands, the command
   dispatcher and result validation, goldens; (a)'s decode of Verse surfaces
   waits on `cultlib-gaps:follow_up:decode-stack-safety`; (b) Parley text, focus, editing and
   IME state; (c) the AccessKit tree.
3. **TUI backend** (Rust): cell measurement in the Taffy measure function,
   terminal drawing, keyboard input. First, because it exercises the whole
   pipeline with no platform bridge and gives agents a reader.
4. **Web backend and shell**: core to wasm, a thin JS host (canvas drawing,
   hidden-textarea IME, DOM accessibility mirror), the discovery client, and the
   web shell that replaces the Nightwing browser reference.
5. **Consumer ports**, one cut per live consumer, each carrying its rename and
   deleting its Thing 1 path (`consumer-port-carries-rename`). Cutover shape per
   `thing2:question:port-cutover`. Proposed order: Odin (reads Thing 2 kinds),
   VoidBot (richest binding, one command), Ghostlight with Heimdall (text inputs,
   the auth effect, the flat-result case; two cuts, Heimdall's access gate
   becomes a composition), Muninn and sleipnir, weksa, Stonks, Vili, Gjallar,
   StreamPixels, then gamecult-ops (the Nightwing target), then CultLib (its eve
   document definitions in `cultcache-ts` and `cultmesh-kotlin` are deleted, B26).
   These need more repos than the campaign lists: `thing2:question:campaign-repo-limit`.
6. **Retirement**: delete Thing 1's tree from the kernel repo. The renames,
   archives and site prose are re-specified as `thing:cut_spec:cut-repo-renames.r2`
   and `cut-site-prose.r2` and do not wait for it (P14).

**Repos this map retires:** EveUnity, EveFlutter, EveElectron, EveTui, EvePlugins
and EveConformance (no Thing 2.0 role: no domain kinds, no per-renderer
lowerers, parity by goldens). They are archived under their Eve names
(`thing2:ruling:retired-repo-names`) by `thing:cut_spec:cut-repo-renames.r2`,
ahead of retirement, because nothing live builds from them (P14). The kernel
repo is renamed Thing in the same cut (`thing2:ruling:code-home`); its URLs in
gamecult-ops stay on the redirect until the port that owns each one (P15).
AetheriaEve stays under its rulings.

**Parked (dormant or blocked) consumers:** Bifrost, repixelizer, Mimir, Hermodr,
Loki, Sai, AquaSynth, Brokkr, Ymir. Their Thing 1 pins keep resolving at the
archived repos; each ports when it is next revived. Unity and Flutter backends
have no live consumer and are parked until one appears.

## Rationale

- **Why the contract before core.** Every later cut, and every consumer port,
  depends on the types; the cross-runtime decode of a recursive union (P9) is the
  riskiest unknown and is cheapest to find before core exists.
- **Why TUI before web.** It has no platform bridge, so the first end-to-end run
  tests core, not a host; the web backend then adds only drawing, IME and the
  accessibility mirror.
- **Why compositions are resolved by core, not expanded by each language's
  builder.** Builder-side expansion would put `metric`, `card` and every shared
  composition in each consumer language, which is Thing 1's per-renderer lowerer
  problem moved to the publishers.
- **Why no expression language in documents.** P5 shows where it leads: the
  renderer becomes a second owner of derived state. Owners publish derived
  values; `Format` covers presentation.
- **Why Odin ports first.** It is the aggregator every viewer discovers
  through; under either cutover option it must read Thing 2 kinds before a
  provider publishes them.
- **Why the result has no free-form payload.** P2 and P7: the only live uses of
  `pluginPayload` and `transientProjection` were a navigation and a status,
  which are `effect` and `outcome`.
- **Why each integer's rule lives in its field type.** The decode door cannot be
  closed (P18), so a rule held only by a constructor is a rule the wire skips.
  A newtype per domain (`Seq`, `Version`, `Count`, `Segment`) is read by both
  paths, keeps the bytes, and gives the TS and C# packages one refusal golden
  each to mirror. This is serde's `try_from` pattern, written by hand because
  the CultCache derive emits the codec itself.
- **Why the key grammar constrains only the local segment.** Provider ids and
  publishers are owned by discovery (`thing2:ruling:advertisement-owner`) and
  are dotted in the Verse today (P19); the contract owns only `label`, `name`
  and `catalogId`. Refusing the separator there makes every key split at its
  last dots, the way DNS labels and the mind's own `<root>:<kind>:<local>` keys
  stay unambiguous. A different separator between provider and label would
  instead impose a grammar on ids the contract does not own. Listing a
  provider's records filters on the record's provider field, not a key prefix,
  because a dotted provider is a prefix of a longer one.
- **Why each sibling holds its domains where its decoder constructs.** The
  rule from contract-rs is that a domain is checked on every door by one owner.
  C# has no newtype that keeps the leaf's bytes without a custom formatter,
  which CultLib owns, but MessagePack, System.Text.Json and code all build a
  C# value through the one constructor that takes the slots (P20), so that
  constructor, calling `ThingDomain`, is the door. cultcache-ts runs each
  slot's codec on encode and decode (P21), so a domain codec over the plain
  leaf is the door there, the same shape as Rust's newtype `Deserialize`.
  `CommandResult` is closed differently in each language: a private
  constructor under `AllowPrivate` in C#, a `#brand` field checked by the
  document's `schema.parse` in TS, because TypeScript's structural types let
  any object with the right fields pass as a class with only public members.
- **Why the siblings read CultLib from a sibling checkout, not a submodule.**
  The C# package already reads CultLib that way (`CultLibRoot`), cultcache-ts
  is on no registry yet (P22), and the Rust crate pins the same commit by
  Cargo rev. A submodule would be a third pin to keep equal to the Cargo rev;
  publication replaces the `file:` path instead.
- **Why contract-rs merges before CultLib's stack fix.** The overflow is the
  decode door's (`cultlib-gaps:follow_up:decode-stack-safety`) and every Rust
  reader of these kinds inherits it whether or not the contract merges. The
  contract decodes no hostile input; core's Verse decode does, so that is where
  the wait sits (cut 2a). The crate docs name the hazard meanwhile.
