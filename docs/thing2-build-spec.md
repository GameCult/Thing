# Thing 2.0 build spec

Status: refreshed 2026-11-10 against the thing2 and thing rulings in force in the
mind (Thing is Persona's stewardship; this spec does not wait on review). Written for Meta's ask t12-m0
(#cultlib, 2026-11-04) on top of `docs/thing2-target.md` (why, invariants) and
`docs/thing2-map.md` (body facts P1-P22, model page, consumer contract, authority
map, cut order). It does not restate them; it adds what the map leaves at
deliverable level: **the pieces, each piece's contract, the build order, and the
test that defends each piece**, with prior art for every design choice.

Nothing here overrides a ruling. Where this spec makes a choice the rulings do not
force, it is labelled **D<n>** (decision, with confidence) and the ones it could
not settle are in "Open" at the end, each with a recommendation.

## How to read the prior-art tags

Each piece ends with **Prior art**. Tags: **[read]** = a source read in an earlier
Eyes or Imagination pass, with its evidence pointer in `docs/research/*` (cited as
R-F = `thing2-foundation-prior-art.md`, R-T = `thing2-text-input-a11y-prior-art.md`,
R-K = `thing2-contract-forks-prior-art.md`, R-C = `thing2-consumer-census.md`);
**[background]** = prior knowledge, not re-fetched for this spec, and the first
thing to verify when its piece is mapped into a `cut_spec`. Confidence is stated
per decision. Meta's rule, as passed to this run, is "prior-art sources cited";
if the rule has more to it (format, depth, a required venue) the Open list asks.

## 0. The machine in one page

```
 provider ──publishes──▶ CultMesh state (typed .cc records)
                              │ StatePort (subscribe / read)
                              ▼
   ┌───────────────────────── thing-core (one crate, internal modules) ─────────────────────────┐
   │ resolve ─▶ bind ─▶ layout ─▶ text ─▶ render ─┐                                              │
   │   (docs,    (Bind,   (Taffy)   (Parley or     ├─▶ FrameOutput { commands, a11y, outbound,   │
   │  compos.)   Repeat)            cells)         │                 ime, diagnostics }          │
   │ input (focus, edit, hit-test, scroll) ─▶ command dispatcher ─▶ outbound CommandInvocation   │
   │ a11y (AccessKit tree)                                                                      │
   └───────────────────────────────────────────────────────────────────────────────────────────┘
        ▲ FrameInput { now, viewport, scale, events }                │ commands + a11y
        │                                                          ▼
     host event loop ◀────── thin backend: TUI (crossterm) | web (wasm + JS) | later hosts
```

Core is a **function of (documents, state, input, time)**. The host owns the loop
and calls `Core::frame`; core never spawns, sleeps, reads a clock or touches a
socket except through a port. This is what makes every test below runnable without
a daemon (`doctrine/gamecult.md`, "dependency injection is boring") and what makes
goldens byte-stable (map rule 10).

**D1. One crate for core, internal modules, not eight crates.** (Confidence
medium-high.) Authority separation (resolve, bind, layout, text, render, command)
is real, but the modules share one frame pipeline and one set of types; a crate per
stage buys build fan-out and a public surface no consumer may use, which
`narrow-consumer-contract` forbids. Splits that *do* earn a crate, each naming the
invariant it protects:

| Crate | Earns its boundary because |
| --- | --- |
| `thing-contract` (landed) | consumers depend on it alone (`narrow-consumer-contract`); three runtimes are held to its goldens |
| `thing-core` | owns geometry, text, focus, binding, results; depends on no transport, no clock, no platform |
| `thing-mesh` | the only crate that depends on CultMesh/CultNet: `StatePort`/discovery implementations. Keeps rule 7 and the mockable-port invariant checkable by a dependency-tree grep on `thing-core` |
| `thing-tui` | backend; depends on `thing-core` output types and `crossterm`, never `taffy`/`parley` (rule 8) |
| `thing-web` | wasm entry (`wasm-bindgen`) plus the JS host package; same dependency rule |
| `thing-testkit` | fake ports, fixture loader, golden harness, shared by every crate's tests; never a dependency of a shipped artifact |

Prior art: Xilem and egui split by renderer/backends and keep one core
(R-K §1: `masonry_core`, `egui`+`epaint`, backends as crates) **[read]**; AccessKit
and Parley are separate repos only where independently consumed or needing another
toolchain (R-K §1) **[read]**; Flutter merged repos to end rolls (R-K §1)
**[read]**. Same repo, many crates, is the dominant pattern.

## 1. Build order and dependency graph

```
 0 decisions ─▶ 1 contract-ts ┐
              ├▶ 1 contract-cs ┴▶ 1x cross-store check
              └▶ 2 testkit+ports ─▶ 3 resolve ─▶ 4 bind ─▶ 5 layout ─▶ 6 render+goldens ─▶ 7 commands
                                                                             │                   │
                                                              8 TUI ◀────────┘                   │
                                              9 text(Parley)/focus/edit ─▶ 10 a11y ─▶ 11 web ◀───┘
                                                                  12 thing-mesh (parallel from 4 on)
                                                                  13 consumer ports (one per consumer)  ─▶ 14 retirement
```

Rules for the order: (a) contract before core (map, rationale); (b) the pipeline is
built **end to end thin first** (cut 8 draws a real surface in a terminal before
text, IME or a11y exist), then deepened, because the first full run is where unknown
unknowns show; (c) `thing-mesh` runs beside core from cut 4 because it is the
other half of the `StatePort` contract and needs the same fixtures; (d) no consumer
port starts before cut 7 and one backend (cut 8) work, so a port can be viewed.

Cuts are sized to Hands' budget (about 150 calls, 0.5M tokens is the danger line):
each row below is one `cut_spec`, or two where marked. Sizes are Self's estimates
in thousand lines of owned code (excluding goldens), pressure not metric.

| # | Cut | Waits on | Est. | Parallel with |
| --- | --- | --- | --- | --- |
| 1a/1b | contract-ts, contract-cs (specs `thing2:cut_spec:cut-contract-ts.r3`, `-cs.r3`) | contract-rs (landed `6ae2da0`) | mapped | each other |
| 1x | cross-store check | 1a, 1b | 0.3 | 2 |
| 2 | testkit + ports | contract-rs | 1.2 | 1a, 1b |
| 3 | resolve | 2 | 1.8 | - |
| 4 | bind | 3 | 1.5 | 12 |
| 5 | layout | 4 | 1.8 | - |
| 6 | render + goldens | 5 | 1.5 | - |
| 7 | commands | 4 (needs 6 only for hit-test) | 1.6 | 8 |
| 8 | TUI backend | 6, 7 | 1.5 | 7 tail |
| 9a/9b | text: measure+shape, then focus+editing+IME state | 6 | 2 + 2 | 8 |
| 10 | a11y tree | 9b | 1.0 | 11 start |
| 11a/11b/11c | web: wasm+drawing, IME mirror + a11y mirror, shell+discovery | 6, 9b, 10 | 1.5+1.5+1.0 | - |
| 12 | thing-mesh | 2, 4 | 1.5 | 5-8 |
| 13.n | consumer ports | 7, 8 (web for browser consumers: 11c) | per consumer | one repo each |
| 14 | retirement | last port | -1x | - |

## 2. Cross-cutting contracts

### 2.1 Core API (the one public surface core has)

```
Core::new(ports: Ports, config: Config) -> Core
Core::frame(&mut self, input: FrameInput) -> FrameOutput
  FrameInput  { now: Instant, viewport: Size, scale: f32, events: Vec<InputEvent>, metrics: MetricsMode }
  FrameOutput { commands: Vec<RenderCommand>, a11y: Option<TreeUpdate>, outbound: Vec<CommandInvocation>,
                ime: Option<ImeRequest>, diagnostics: Vec<Diagnostic>, next_wake: Option<Instant> }
Ports { state: dyn StatePort, fonts: dyn FontPort, clock: dyn Clock, ids: dyn IdSource, host: dyn HostEffects }
```

- **One owner for time and ids.** `now` arrives in the frame input; `IdSource`
  mints invocation ids. Tests substitute both, so a run is a pure function and
  invocation ids in goldens are reproducible.
- **Outbound is invocations only** (rule 7). `Ports.state` has `read`, `subscribe`,
  `list(schema, key_prefix)`; it has no `write`. The one write path is
  `FrameOutput.outbound`, which the host or `thing-mesh` forwards.
- **`HostEffects`** has exactly the closed `effect` set of the contract:
  `open_surface(key)`, `open_external(url)`; plus `clipboard` read/write when the
  text piece needs it (see Open O7).
- Metrics mode (`Pixels` | `Cells`) is a core input, because the TUI measures in
  cells inside the Taffy measure function (R-T §5: iocraft does exactly this
  **[read]**) and rule 8 forbids the backend from owning any layout input. The
  cell-metrics provider is part of core (small: `unicode-width` and
  `unicode-segmentation`), not of `thing-tui`.

Prior art: Clay's per-frame contract (set dimensions → pointer state → begin layout
→ end layout returns the command array) is the model for `frame` **[read, R-F
§2.1]**; egui and Bevy UI emit shapes or commands to thin backends **[read, R-F
§2.4, R-T §1]**. Elm/Redux style "update is a pure function of state and event"
**[background]**. Confidence high: no contrary evidence found.

### 2.2 The closed render-command list (D2)

`FillRect{rect, radius, color}`, `StrokeRect{rect, radius, width, color}`,
`Text{origin, run, color}` where `run` is `Glyphs{font, size, glyphs}` (Pixels mode)
or `Cells{col, row, text, style}` (Cells mode), `Image{rect, catalog, asset, fit}`,
`ClipPush{rect, radius}`, `ClipPop`, `Polyline{points, width, color}`,
`Polygon{points, color}` (plot marks), `Embed{rect, slot_kind, slot_id}`.
Nine variants. Selection highlights, carets, focus rings and scrollbars are
`FillRect`/`StrokeRect` that core emits. Z order is emission order; there is no
z-index, no layers, no custom escape. A new command is a reviewed change to core
plus every backend and the goldens at once (that is the point of closing it).

Colours in commands are resolved literals; theme tokens never cross to a backend.
Coordinates are `f32` logical pixels in Pixels mode, integer cells in Cells mode.

Prior art: Clay's complete enum is `NONE, RECTANGLE, BORDER, TEXT, IMAGE,
SCISSOR_START, SCISSOR_END, OVERLAY_COLOR_START/END, CUSTOM`, with renderers for
SDL, raylib, canvas2d, termbox2 and a terminal, all thin **[read, R-F §2.1]**.
Thing's list differs on purpose: no `CUSTOM` (Embed is the closed host slot), no
overlay-colour pair (an overlay scrim is a `FillRect`), plus polylines for `Plot`.
Confidence medium: the count is a proposal; whether `Plot` should lower to
`Polyline`/`Polygon` or be left to an `Embed` is O3.

### 2.3 Diagnostics (D3)

A closed `Diagnostic { code, node, document, detail }`: `UnresolvedUse`,
`BindSchemaMismatch`, `BindMemberMissing`, `UnknownNode`, `UndeclaredCommand`,
`DuplicateNodeId`, `StaleVersionIgnored`, `MalformedResult`, `UnknownInvocation`,
`InvocationTimedOut`, `DecodeRefused`. The same value drives (a) the placeholder or
error node drawn in the frame, (b) a log line, (c) the a11y tree (an error node is
announced), and (d) the tests. Rules 2, 4, 5 and 6 are asserted on these codes, so
a rule that is deleted fails by name, not by a screenshot.
Confidence high; the alternative (free-text logs) is what Thing 1's gap ledgers were.

### 2.4 Test layers and golden format (D4)

Doctrine splits tests into local-authority unit tests, adjacent-organ pipeline
smokes, and end-to-end Verse contract runs; every piece below names which layer.

- **Fixtures** (`tests/fixtures/*.surface.cc`, committed): typed documents encoded
  through `thing-contract`, one per row of the fixture table (§2.5). Built by a
  `thing-testkit` builder in Rust, then frozen; the bytes are what is loaded.
- **Geometry golden**: for a fixture + viewport + state script, the resolved node
  tree with `(node id, x, y, w, h)` in a stable text form.
- **Render golden**: the `Vec<RenderCommand>` encoded with CultCache MessagePack
  through the same derive the contract uses, plus a line-oriented text rendering
  beside it for diffs. The binary form is the oracle (byte-identical, rule 10); the
  text form is only for review.
- **Regeneration** is `cargo xtask bless`, runs only on a reviewed core change,
  and a backend crate cannot call it (model page: goldens are the core's).
- **Layout correctness is inherited, tree building is tested.** Taffy is itself
  validated against Chrome with generated fixtures (`just gentest`, R-F §2.2
  **[read]**), as is Yoga (`gentest/`, R-F §2.3 **[read]**); Thing does not re-prove
  flexbox. Thing's tests prove that *our* style mapping puts the right inputs into
  Taffy and that the output flows into commands unchanged. One extra check
  re-derives a small fixture's geometry from the raw Taffy API directly and compares
  (guards against our mapping drifting while goldens are re-blessed).
- **Mutation**: `tools/cargo-mutants.sh` once per Hands pass, scoped to the cut's
  diff, per the Eureka rule. Each piece's "Test" below names the rules whose
  mutants must die; a survivor is triaged, not hidden.
- **Property tests** (`proptest` [background]) only where a total function has a
  clear law: contract round-trip (exists), `resolve` never panics and either
  returns a tree whose ids are unique or a refusal, `bind` is idempotent for the
  same state, `frame` with no events and unchanged state returns an equal command
  list (stability). Fuzzing the decoder (`cargo-fuzz` [background]) waits only on Eve taking the
  CultLib pin that carries `decode-stack-safety` (Fixed, CultLib `bbb98eaf`,
  `cultlib-gaps:resolution:follow_up.decode-stack-safety.n1`; map P20).

Prior art: golden command lists are not published by any project read (R-F §2.5:
"not found, not searched exhaustively"), so this is the least-supported decision
in the spec; the nearest supported techniques are Taffy/Yoga generated geometry
tests and egui's `egui_kittest` snapshot crate (R-K §1 lists the crate;
its snapshot method is **[background]**). Confidence medium: the idea is sound
because commands are renderer-independent by construction; cost of churn on
intentional change is the risk, mitigated by `bless` being one command.

### 2.5 Fixture set (the census sizes the floor)

Twelve fixtures cover the node set and every rule; each is named for the consumer
shape it models (R-C §7 cross-table is the source, and Thing is a general library,
so a fixture exists for a capability a reasonable consumer would expect even with
no caller today):

| Fixture | Exercises | Rules |
| --- | --- | --- |
| `f01-swarm-dashboard` | VoidBot-shaped: Box grid, bound Text/Format, `Repeat` over a collection prefix, one Button | 6, 9 |
| `f02-login-effect` | Ghostlight/Heimdall: TextInput, Button, `OpenExternal{resume}` result | 1, 2 |
| `f03-input-target` | Muninn: `position: absolute` target over an `Image`, bound label (P6) | - |
| `f04-form` | Toggle, Slider, Select, TextInput number, `captures` and `clear_drafts` | 1 |
| `f05-modal-overlay` | `Overlay` modal with dismiss command, focus trap | - |
| `f06-scroll-list` | `overflow: scroll`, `Repeat` 1k items, keyed insert/remove | - |
| `f07-plot` | Stonks/StreamPixels-shaped `Plot` series | - |
| `f08-embed-viewport` | `Embed` slot as a leaf of given size | - |
| `f09-composition-stack` | nested `Use` with typed args and child slots, `thing.std` plus a publisher composition | 5 |
| `f10-hostile` | unknown arm, undeclared `CommandRef`, duplicate id after expansion, unresolved `Use`, schema-mismatched `Bind` | 4, 5, 6 |
| `f11-deep` | nesting at the 256 bound (decoded on a large-stack thread until Eve's pin of CultLib `bbb98eaf` merges) | - |
| `f12-text-edge` | bidi, CJK wide characters, combining marks, emoji ZWJ, long wrap | - |

## 3. The pieces

Each piece lists **Owner** (one writer), **In**, **Out**, **Ports**,
**Invariants** (what must die under mutation), **Failure**, **Test** (layer and the
rule each pins), **Deletes/forbidden** (what the piece must not contain), and
**Prior art**. Map rules are cited as M1-M10 (the ten in the map's "Rules that must
die under their own mutation").

### Piece 1: contract (packages for TS and C#, and the store check)

Rust (`crates/thing-contract`) landed after two Soul passes; `contract-ts` and
`contract-cs` are mapped (`thing2:cut_spec:cut-contract-ts.r3`, `-cs.r3`; r2
is superseded) and have
branches (`hands/thing2-contract-ts`, `hands/thing2-contract-cs`) not yet merged.
This spec adds no contract change. Rulings that bind the packages:

- **Unknown union arms** are refused by CultCache in every runtime, the C#
  reference included, and Thing packages carry no check of their own
  (`thing2:ruling:contract-decode-tolerance`; the fix is CultLib's, follow-up
  `thing2:follow_up:cultcache-union-parity`, resolved in the mind). The "unknown union arm is refused"
  test below therefore exercises CultCache, not package code.
- **Nil in required slots** is refused by each package: constructors and factories
  reject null, decoding nil into a slot the Rust contract does not make optional
  is a typed refusal (`thing2:ruling:contract-refuses-nil-in-required-slots`,
  Defaulted). In TS this is a `present(codec)` on every non-Option slot, and
  results are validated and frozen at one factory (`cut-contract-ts.r3`).
- **Byte payloads** (`RecordPayload.Bytes`) are a copied read-only view in C#
  (`ReadOnlyMemory<byte>`); Rust's `Vec<u8>` and the wire are unchanged
  (`thing2:ruling:payload-bytes-read-only`, Defaulted). The catalog type name for
  that slot changes with it, which the 1x store check must read from the C# set.

- **Owner:** the contract crate; packages own nothing (map authority).
- **Test (per package):** the shared golden fixtures decode and re-encode
  byte-identical (M1, M3); the refusal goldens (`seq` 0, extra slot, unknown union
  arm) are refused; a hand-built object with seq 0 is refused at encode.
- **1x cross-store check:** the C# package opens a `.cc` store the TS package wrote
  and the reverse, and both open Rust's. The check lives in `thing-testkit` as a
  scripted cross-runtime job run on Yggdrasil's verify stopgap
  (`tools/stopgap/ygg-verify.sh`), because only that image has all three runtimes.
  It proves the catalog type names agree (map cut 1 waiting line).
- **Prior art:** CultCache wire parity against the C# reference is the standing
  CultLib rule; probe P12-P13 are the evidence **[read, map]**; golden-file
  interoperability is how Protobuf and Cap'n Proto test language bindings
  (`conformance` suites) **[background]**. Confidence high.

### Piece 2: testkit and ports

- **Owner:** `thing-testkit` owns fakes; `thing-core` owns the port *traits*
  (a trait is the narrow port; the fake is a test crate's business).
- **In/Out:** `StatePort { read(schema,key)->Option<Entry>, list(schema,prefix),
  subscribe(schema,key|prefix) -> Stream<Change> }` where `Change` is
  `Upsert(entry)|Delete(key)`; `FontPort { resolve(family, weight, style) ->
  FontHandle, bytes(handle) }`; `Clock`; `IdSource`; `HostEffects`. All object-safe.
- **Fakes:** `MemState` (a `CultCache`-backed map with a scripted change feed and
  a record of every call), `FixedFonts` (one bundled test font, so Parley output is
  deterministic across machines), `ManualClock`, `SeqIds`.
- **Invariants:** `StatePort` has no write method (M7, checked by the trait's
  method list in a test that enumerates it and by a grep over `thing-core`);
  `MemState` returns entries only as decoded contract records, never `Value`.
- **Failure:** a decode refusal from the port surfaces as a `Change::Refused`
  carrying the code, never as a missing record (a refusal must not look like
  absence).
- **Test:** unit. `MemState` change ordering; fixture loader round-trips all twelve
  fixtures through the contract (a fixture that fails the contract is a bug in the
  fixture before it is a bug in core).
- **Deletes/forbidden:** no global state, no `Instant::now()` in core (negative
  grep).
- **Prior art:** the port style is the project's own doctrine; Xilem's
  `masonry_testing` and egui's `egui_kittest` are test-harness crates beside a core
  (R-K §1) **[read]**. Confidence high.

### Piece 3: resolve

- **Owner:** `thing-core::resolve`. It is the only code that interprets Thing
  Script structure.
- **In:** `SurfaceDocument`, a `CompositionStore` (std compiled in, plus publisher
  compositions read through `StatePort`). **Out:** `ResolvedTree` (arena of
  nodes, unique ids, every `Use` expanded or replaced by a placeholder node, every
  `CommandRef` checked against the document's `CommandDecl`s, bindings still
  unresolved), plus diagnostics.
- **Invariants:** M4 (unknown kind, undeclared `CommandRef`, duplicate id after
  expansion refuse the document *whole*, and the old document stays displayed);
  M5 (unresolved `Use` is a visible placeholder with a diagnostic); expansion is
  substitution only (no conditionals, no arithmetic: a test that every
  `thing.std` composition expands under a parameter-less evaluator); expansion
  terminates (recursion in compositions is refused with a cycle diagnostic and a
  depth bound); expansion is deterministic and order-preserving; ids of expanded
  nodes are derived from the `Use` node id and the template id by one scheme, so
  they are stable across republishes (focus and scroll state key on them).
- **Failure:** a referenced composition arrives later → the placeholder is
  replaced next frame without a republish.
- **Test:** unit per rule on `f09`, `f10`; property (`resolve` total, ids unique or
  refusal); pipeline smoke: `resolve` output fed to `layout` (cut 5) in `f01`.
  Mutants that must die: skip the duplicate-id check; treat an undeclared
  `CommandRef` as a warning; drop the placeholder; expand `Use` without pinning the
  composition version.
- **Deletes/forbidden:** no `kind` string dispatch anywhere (closed enum only); no
  fallback along a kind path (M4); no expression evaluator. No text-syntax parser
  outside `thing-core` (`thing-script-form`): `resolve` consumes only the typed
  document; a later `.thingscript` parser is core's and feeds it.
- **Prior art:** Adaptive Cards separates template expansion from render
  (`adaptivecards-templating` expands a template against data before render)
  **[read, R-K §2]**; Thing's `Use` is the same split without an expression
  language, which R-K §2/P5 shows the failure of (the DSL's renderer-local `derive`)
  **[read]**. Slint compiles a text language to a typed tree at build/run time
  **[read, R-K §2]**. Confidence high on substitution-only; medium on id-derivation
  (no source read; it is the same problem React solves with keys **[background]**).

### Piece 4: bind

- **Owner:** `thing-core::bind`. Only code that reads typed state.
- **In:** `ResolvedTree`, `StatePort`. **Out:** `BoundTree` with every `Bind`
  replaced by a typed value or a binding-error value; `Repeat` expanded to one
  subtree per item, keyed by the item key; subscriptions established and released
  as bindings appear and disappear.
- **Invariants:** M6 (different schema or missing member renders a typed error,
  never a coerced value); M9 (a version at or below the held one is ignored, and
  the ignore is a `StaleVersionIgnored` diagnostic at most once per document
  version, not per frame); a value change never triggers layout of anything but the
  nodes whose bound values changed (dirty tracking is observable through a counter
  the test reads); `Format` is total and locale-free in core (formats are the
  closed set; any locale is a host input, see O8); `Repeat` item keys must be
  unique, a duplicate key refuses the document at bind time as M4 does; keyed
  reorder preserves node identity, so focus and scroll survive a reorder.
- **Subscription rule:** core subscribes to exactly the `(schema,key)` pairs and
  prefixes the live tree references; a document replacement diffs its subscription
  set. An unsubscribed record's arrival changes nothing.
- **Failure:** record absent → a typed `Absent` value rendered as the node's
  declared empty text (the contract field), not an error; schema mismatch → error.
- **Test:** unit per rule on `f01`, `f06`, `f10` with scripted `MemState` feeds;
  pipeline smoke `bind → layout` checks dirty-node count; mutants: coerce a
  mismatched schema; drop the version gate; resubscribe every frame; reorder by
  index instead of key.
- **Deletes/forbidden:** no local `var`/`derive`/`collection` store (P5); no draft
  binding (BD2: editing state is `input`'s).
- **Prior art:** BD1-BD9 collapse in the map **[read, map]**; keyed list
  reconciliation is React's `key` and Elm's `Html.Keyed` **[background]**;
  Clay retains element state by ID and requires `CLAY_IDI` for loops (R-F §2.1)
  **[read]**, the same constraint. Flutter's retained tree keeps state by key
  **[background]**. Confidence high.

### Piece 5: layout

- **Owner:** `thing-core::layout`. The only crate-user of `taffy`.
- **In:** `BoundTree`, viewport, metrics mode, `FontPort`. **Out:** `LaidTree`:
  per node a content box, scroll extents, clip, and absolute-positioned overlay
  boxes.
- **Contract:** the mapping from contract `BoxStyle` to Taffy style is one
  function with an exhaustive match (a new contract field fails to compile until it
  is mapped). Leaf measure for `Text`, `TextInput`, `Image`, `Embed`, `Plot`
  goes through one `Measure` trait: Pixels → Parley (piece 9), Cells → cell metrics,
  `Image`/`Embed`/`Plot` → declared sizes. Overlay nodes are laid out as a
  separate root attached to an anchor node's final rectangle or the viewport.
  Scroll containers expose `content size` and clamp offsets that `input` owns.
- **Invariants:** layout is a pure function of (tree, viewport, scale, metrics,
  fonts): same inputs, byte-identical geometry (M10); a style field with a literal
  and a token resolves via the one theme resolver before reaching Taffy; `fr`,
  `%`, `auto` and `content` sizes reach Taffy as the matching Taffy dimensions
  with no rewriting; an `Embed` is a leaf of exactly its declared size.
- **Failure:** NaN or negative sizes from a document are refused at resolve (the
  contract already refuses at decode for `Count`); layout never panics on a
  constraint it cannot satisfy and instead overflows per CSS rules.
- **Test:** unit geometry goldens on `f01`, `f03`, `f05`, `f06`, `f08`; the
  raw-Taffy rederivation on one small tree (§2.4); property: layout is
  deterministic across repeats and scale 1 vs 2 scales positions linearly when
  fonts are scale-independent (fixed font); Cells mode: `f12` widths equal
  `unicode-width` expectations. Mutants: swap row/column mapping; drop `gap`;
  round positions; ignore `position: absolute` insets.
- **Deletes/forbidden:** `taffy` appears in no other crate's tree (M8, `cargo tree
  -i taffy` negative check on `thing-tui`, `thing-web`, `thing-mesh`, contract).
- **Prior art:** Taffy implements Block, Flexbox and Grid, is used by Servo, Blitz,
  Bevy, Slint, iocraft, Zed/GPUI and is validated against Chrome (R-F §2.2) **[read]**;
  measure-function leaf text is the documented integration point (R-F §2.2, R-T §1)
  **[read]**; iocraft's measure for Cells (R-T §5) **[read]**. Ruling
  `thing:ruling:thing2-one-solver-taffy`. Pin: Taffy 0.14.0 (2026-08-24 [read]);
  Slint pins 0.10 flexbox-only and iocraft 0.5.2 (R-T §1, §5), so the Grid feature
  surface at 0.14 is the one to prove first (confidence medium).

### Piece 6: render and goldens

- **Owner:** `thing-core::render`. Turns `LaidTree` + paint + input state into the
  command list; draws selection, caret, focus ring, scrollbars.
- **Invariants:** M10 (byte-identical command list for equal inputs); painter
  order is document order, then overlays last; `ClipPush`/`ClipPop` balance (a
  validator over every golden); every `Embed` and `Image` node produces exactly
  one command or a visible missing-asset placeholder (diagnostic `AssetMissing`
  added to D3's list when this cut lands, O9); nothing outside the clip stack is
  drawn for `overflow: clip|scroll` nodes.
- **Backends must prove consumption (M10 second half):** each backend's parity
  test replays a golden list and asserts one draw call per command; a backend that
  skips a command variant fails by name.
- **Test:** golden suite described in §2.4 over all fixtures; pipeline smoke
  `resolve→bind→layout→render` on `f01`; mutants: drop `ClipPop`, reorder overlay,
  emit caret when unfocused, emit a command with a theme token not a literal.
- **Prior art:** Clay's sorted command array and scissor start/end (R-F §2.1)
  **[read]**; egui/epaint clipped shapes to thin backends (R-F §2.4) **[read,
  background]**. Confidence high on structure, medium on exact variants (O3).

### Piece 7: commands (dispatcher and result validation)

- **Owner:** `thing-core::command`. Mints invocations, tracks them, validates
  results. The only producer of `FrameOutput.outbound`.
- **Lifecycle (D5):** `Issued → Pending* → Completed | Denied | Failed`, with two
  core-only terminal states, `Malformed` (result failed to decode, or named an
  unknown invocation) and `TimedOut`. Terminal states are final: a later result
  for a terminal invocation is a diagnostic, not a state change.
- **In:** input events that activate a `CommandRef` node; results arriving as
  `StatePort` changes of schema `gamecult.thing.command_result.v1`.
  **Out:** `CommandInvocation` (id from `IdSource`, `captured` from `input`'s editing
  state for the nodes `CommandDecl.captures` lists, `surface` = key + held version,
  `issued_at` from the clock; **no caller identity field written**, the transport
  sets it), UI state for the issuing node (busy, denied-at-node), effects through
  `HostEffects`.
- **Invariants:** M2 (a result that fails to decode, or an unknown `invocation_id`,
  becomes a visible `Malformed` state for that invocation and a log line; no
  invocation stays pending past its timeout without one); M7 (core writes nothing
  but invocations); idempotency: one user action mints exactly one id, and a
  re-delivered invocation carries the same id; `Completed{clear_drafts:true}`
  clears exactly the captured nodes' drafts and no others; `OpenExternal{resume}`
  is executed once per result `seq`; a `Denied{node}` marks that node and no
  other; a double-click on a busy Button mints no second invocation while the first
  is non-terminal (the Button node's enabled state is derived from lifecycle, not a
  flag a document sets).
- **Timeout:** a core config value, default 15 s in `Config` (O5: it is a guess).
- **Failure:** `StatePort` refusal on a result (piece 2's `Change::Refused`) maps to
  `Malformed`, which is the case P2 shows Thing 1 lacked (result rejected whole
  by `ajv` with nothing visible).
- **Test:** unit lifecycle table (every transition, every illegal one), driven
  with `ManualClock` for timeouts; pipeline smoke with `MemState`: press a Button in
  `f02`, assert exactly one outbound invocation with the right captures, feed
  `Pending` then `Completed{OpenExternal}`, assert one host effect; negative:
  result with extra field (M1) never reaches the lifecycle. Mutants: no timeout;
  mint id per frame; `clear_drafts` clears all; accept results after terminal;
  write caller identity.
- **Prior art:** idempotency keys for commands and retries (Stripe API
  idempotency, RFC draft "Idempotency-Key" header) **[background]**; command/result
  split with a pending phase (job/operation resources in Google AIP-151 long-running
  operations) **[background]**; Flutter's `TextInputConnection` shows what drafts the
  host must hold (R-T §4) **[read]**. Evidence of the failure it prevents: P2, P7
  **[read, map]**. Confidence medium-high; the `Malformed` rule follows directly
  from P2.

### Piece 8: TUI backend (`thing-tui`)

- **Owner:** `thing-tui`: terminal drawing, keyboard and mouse into `InputEvent`.
  First end-to-end consumer of core; also the agents' reader.
- **In:** `FrameOutput.commands` in Cells mode. **Out:** terminal bytes via
  `crossterm`; `InputEvent`s.
- **Invariants:** M8 (no `taffy`/`parley` in the tree); every `RenderCommand`
  variant is handled and each is either drawn or explicitly declared
  not-representable in cells (`Image` → block placeholder with alt text,
  `Polyline` → braille or block approximation): "not representable" is a table in
  the crate, tested; wide and zero-width characters advance the cursor as the
  `Cells` run says (the backend never re-measures); the screen diff writes only
  changed cells (testable on a `Vec<Cell>` buffer without a terminal).
- **Test:** unit on a virtual screen (`ratatui`-style test buffer is
  **[background]**; ours is a `Vec<Cell>`): every golden command list renders to a
  stored screen dump; pipeline smoke: `f01` through the whole of cuts 3-7 to a
  screen dump; parity (M10): every command variant produces a cell effect.
  Mutants: skip `ClipPop`; mis-advance on a wide char; draw a command outside the
  clip; ignore `Text` colour.
- **Forbidden:** layout of any kind.
- **Prior art:** Clay has `termbox2` and terminal renderers as thin command
  consumers (R-F §2.1) **[read]**; iocraft is a Taffy-based TUI (R-T §5) **[read]**;
  ratatui uses Cassowary not Taffy and pins `unicode-width >= 0.2.0` (R-T §5)
  **[read]**, a version issue to carry into our own pin (confidence medium: width
  tables differ between 0.1.x and 0.2.x, so the cell widths in goldens must be
  pinned to a crate version, D6).

### Piece 9: text (9a measure+shape, 9b focus+editing+IME state)

- **Owner:** `thing-core::text` (`thing:ruling:thing2-thing-owns-text`, operator:
  core owns shaping (Parley), editing and IME state, focus and the AccessKit tree;
  backends only draw and bridge to platform protocols; glyph rasterization stays
  backend-private and the render-command contract carries shaped glyph runs).
- **9a In/Out:** string + `TextStyle` + width constraint → `TextLayout` with
  line boxes and glyph runs (Pixels, via Parley) or cell lines (Cells). One
  `Measure` implementation per metrics mode behind the layout `Measure` trait.
- **9b In/Out:** `InputEvent`s targeted by hit-test/focus → editing state per
  `TextInput` node (value, selection, composition range), caret rectangle,
  `ImeRequest { rect, text_window, selection }` (the data a hidden host element
  needs), and edits fed to `command` as captured values.
- **Focus (D7):** one focus owner in core. Tab order is document order of
  focusable nodes, overlays trap focus while modal, focus is keyed by node id and
  survives republish when the id survives, and a focused node removed moves focus
  to its previous sibling then parent. A backend never owns focus; it forwards host
  focus loss/gain only.
- **Invariants (9a):** shaping and bidi never run in a backend (M8); a run's glyph
  positions are a pure function of (text, style, width, font bytes); wrap and
  `max_lines` are applied by core and appear in the command list.
- **Invariants (9b):** the draft is core's state and dies with the session (model
  page row 5); composition text is displayed but not committed or captured until
  the host commits; an IME update never rewrites committed text; every selection
  edit is on grapheme-cluster boundaries (`unicode-segmentation`); `ImeRequest`
  windows the text rather than mirroring a whole large value, and coalesces to one
  per frame (R-T §2: GPUI's mirror coalesces writes because each write restarts the
  IME connection **[read]**).
- **Test:** 9a: unit on `f12` (bidi order, CJK width, combining marks, ZWJ
  emoji clusters, wrap at several widths) with the bundled font; golden glyph
  runs. 9b: scripted event sequences (type, move, select, delete across a
  cluster, compose-update-commit, compose-cancel, undo if D8 includes it) assert
  value/selection/`ImeRequest` at each step; focus: traversal order on `f04`,
  trap on `f05`, removal on a republish. Mutants: byte-index deletion in a
  cluster; commit composition on update; focus not keyed by id; mirror whole value.
- **Prior art:** Parley: rich-text layout with HarfRust shaping, ICU4X bidi and
  segmentation, `PlainEditor`, IME primitives, an AccessKit editor example
  (R-T §1) **[read]**; Bevy (taffy 0.14 + parley 0.11), Slint (`shared-parley`),
  Blitz (Stylo + Taffy + Parley, text input state in the DOM node), Masonry (R-T §1-2)
  **[read]**: four independent Taffy+Parley integrations. Cosmic-text is the
  alternative; Parley wins on the integrations above and AccessKit ties, not on
  measured quality (R-T §1; no benchmark read). IME-restart and mirror behaviour
  from GPUI web, egui `text_agent`, Flutter web `text_editing.dart` (R-T §2)
  **[read]**. Confidence medium: all of this is an assembly of read-but-not-built
  parts; the biggest risk is Parley under `wasm32` and `no_std`-adjacent builds
  (not established; O1). Slug/Vello GPU glyphs (R-T §6) is the operator's scope
  addition and is **out of this build order**: glyphs reach the backend as
  positioned runs, and how the web backend rasterizes them (canvas 2D first,
  GPU later) is O2.

### Piece 10: a11y tree

- **Owner:** `thing-core::a11y`. Derives an AccessKit `TreeUpdate` per frame from
  the resolved tree, roles, labels, focus, text editing state and layout.
- **Invariants:** every focusable node has a role and an accessible name (a
  document that offers none gets the diagnostic `NoAccessibleName`, O9); roles come
  from the contract's closed `Role` set mapped by one exhaustive match; bounds equal
  layout bounds; updates are incremental (unchanged nodes omitted), keyed by node
  id; an `Embed` is exposed as a generic node with its label; text input exposes
  value, selection and composition as AccessKit text-run data.
- **Test:** unit: tree for `f01`, `f04`, `f05` compared to stored AccessKit
  dumps; incremental update test (change one bound value → exactly the changed
  node plus ancestors needed); mutants: drop `focus`; stale bounds after scroll;
  role map swap. An actual screen reader cannot be run in CI; it is an operator
  check at web cutover (O6).
- **Prior art:** AccessKit as the schema, with Bevy, Blitz, Masonry, Slint, GPUI
  and egui integrating it (R-T §3) **[read]**; Blitz's `build_accessibility_tree`
  tests (`accessibility_roles.rs`) are the closest test style **[read]**. Web
  adapter: **planned only** in AccessKit (R-F §2.6, R-T §3), so the web backend
  mirrors the tree to DOM itself (piece 11b); Flutter web is the shipped precedent,
  and its open issues (semantics DOM stealing taps, R-T §3) are the failure
  catalogue for 11b. Confidence high on native, medium on web.

### Piece 11: web backend and shell (`thing-web`)

- **Owner:** `thing-web` (wasm entry + a small JS host package). Replaces the
  Nightwing browser reference (`nightwing-eve-browser-reference`, P8).
- **11a draw:** core compiled to wasm, `frame` called from `requestAnimationFrame`
  with batched events; commands drawn to one canvas 2D context (D9: canvas 2D
  first, because GPUI web already rasterizes through OffscreenCanvas 2D, R-T §2,
  and the contract is already glyph-run based so a GPU path swaps in under the same
  commands). The JS host holds **no layout, no text shaping, no focus state**.
- **11b IME + a11y:** a hidden `<textarea>` placed from `ImeRequest` (the pattern
  all three of egui, GPUI and Flutter web converge on, R-T §2), mirrored text
  windowed and coalesced; a DOM accessibility mirror of the AccessKit update,
  clipped so it cannot intercept pointer events (the exact Flutter failures in R-T
  §3: #194011, #192466), off by default until an assistive-technology signal or an
  explicit toggle, as Flutter does.
- **11c shell + discovery:** a web app that discovers surfaces through the Verse's
  discovery owner (map: discovery is not Thing's, `advertisement-owner`; until the
  owner's typed advertisement lands, `thing2:follow_up:discovery-owner-campaign`,
  through one adapter over today's `gamecult.eve.provider_advertisement.v1`,
  deleted when it does) and mounts core; reachable by `thing-mesh`'s transport or a gateway (O4).
- **Invariants:** M8 (no `taffy`/`parley` in the JS package or the wasm host
  glue); the JS host calls only `Core::frame`, passes only the closed
  `InputEvent`s, and draws only the closed commands; the hidden textarea never
  scrolls the page into view (R-T §2: egui's park-over-canvas) and never receives
  pointer events; one frame's work for `f06` (1k items) stays under a stated budget
  (O10: a number is owed, none was measured).
- **Test:** unit in `wasm-bindgen-test` (headless Chrome and Firefox in the verify
  image): replay goldens through the canvas and compare against a canvas-drawn
  reference of the same list (command consumption, M10); IME: scripted composition
  events at the textarea produce core edits equal to the 9b scripts; a11y: DOM
  mirror equals the AccessKit tree for `f04`; click-through: a pointer test over a
  mirror region reaches the canvas target (guards the Flutter failure). Real IME
  and real screen readers are operator checks (O6).
- **Prior art:** R-T §2-3 as above; EditContext is Chrome-only (MDN, R-T §3) so it
  is not the base path; `egui` web and Flutter web **[read]**; Slint web has no
  accessibility and is not recommended for general web apps (R-T §2) **[read]** (the
  failure this piece exists to avoid). Confidence medium: no Taffy+AccessKit
  web combination exists in the sources read (R-T Gaps), so 11b is first of its
  kind for this stack.

### Piece 12: thing-mesh (the CultMesh adapters)

- **Owner:** `thing-mesh`: `StatePort` and a discovery client over CultMesh/CultNet
  and CultCache (the client is a port: Thing's contract has no advertisement of its own, `advertisement-owner`); the outbound invocation sender; the only crate with a transport
  dependency.
- **Invariants:** a `StatePort` never returns `serde_json::Value` or an untyped
  blob (P3); decode runs through the contract types and refuses on error
  (`Change::Refused`), never a silently-dropped update; subscription uses the mesh's
  own mechanism, never a poll loop (BD9 dies); the sender sets the caller identity
  from the transport session and has no API to override it; decode needs no special thread once Eve pins CultLib `bbb98eaf`, which
  carries `cultlib-gaps:follow_up:decode-stack-safety` (Fixed); until that pin
  merges, decode runs on a thread with the stack it needs (P20).
- **Test:** pipeline smoke: a real CultMesh in-process loopback publishes
  `f01`'s state and a document; core with `thing-mesh` shows it; then a hostile
  record (unknown arm) yields `Change::Refused`; an invocation sent through the
  sender arrives at a fake provider with the transport's identity and not any value
  the core supplied. End-to-end (per Verse contract): later, with Odin's discovery.
- **Prior art:** the doctrine itself (CultMesh is the state substrate;
  `doctrine/gamecult.md`); service registries are not owned by the UI toolkit
  (R-K §3: Consul, Kubernetes Services **[background]**; Teams manifests vs cards
  **[background]**). Confidence high on shape, unknown on the CultMesh API surface
  it must wrap: that API is read when the cut is specced (O4).

### Piece 13: consumer ports (template)

One `cut_spec` per consumer, each carrying its rename (T, E1-E7 in
`thing-campaign.md`) and deleting its Thing 1 path in the same cut. Order per map
cut 5 (Odin, VoidBot, Ghostlight+Heimdall, Muninn/sleipnir, weksa, Stonks, Vili,
Gjallar, StreamPixels, gamecult-ops, CultLib), with cutover shape per
`thing2:ruling:port-cutover`: live consumers cut over one at a time, never all at
once.

Rulings that shape every port:

- **Naming and wire.** The eve-to-thing wire rename is frozen and folded into Thing
  2.0; no wire, window or prose rename lands apart from a port. Thing 2.0 adopts
  rename table T and edge table E1-E7 as its naming rule, so surviving kinds are
  born `gamecult.thing.*` (`thing:ruling:wire-rename-freeze-fold`).
- **No compatibility ids.** No `CompatibleSchemaIds` on any renamed type. An
  Eve-typed store is deleted and rebuilt by its daemon, conformance packs are
  re-exported under thing ids, and a store found to be truth stops its cut as a
  fork (`thing:ruling:operator-compat-rebuild`). A port therefore carries a
  rebuild step, not a reader of old ids.
- **No new Eve surfaces.** Thing, formerly Eve, is deprecated; every new surface
  waits for Thing 2.0 (`thing2:ruling:surfaces-wait-for-thing2`). Operator
  dashboards are quarantined (parked tag, removed from main and the build) and
  rebuilt on Thing 2 by follow-ups, e.g. `thing2:follow_up:ymir-operator-view-on-thing2`,
  `weksa-`, `mimir-`, `muninn-operator-view-on-thing2`
  (`thing2:ruling:eve-dashboards-quarantined-2`). Product UIs are decided case by
  case: `Mimir.EveSensorReceiver` and the Pitch page's visual-novel component are
  frozen until Thing 2, neither extended nor removed
  (`thing2:ruling:mimir-sensor-receiver-frozen`, `thing2:ruling:pitch-component-frozen`).
- **Discovery is not Thing's.** A port reads the provider advertisement through the
  `thing-mesh` port, not through a Thing-owned kind (`thing2:ruling:advertisement-owner`).
- **Repo set.** The code home is the Eve repo renamed Thing; Unity and Flutter
  backends get their own repos only when a live consumer needs one
  (`thing2:ruling:code-home`). The campaign waits for `cut-campaign-repos`, then
  target r2 names the 13 repos before the first port
  (`thing2:ruling:campaign-repo-limit`). The six retired family repos are archived
  under their Eve names (`thing2:ruling:retired-repo-names`).

**Port-list gaps.** Two consumers are in none of the lists above. (1) gamecult-site
generates a Thing 1 surface on `gamecult.eve` ids (`gamecult-compound.eve-gui.json`,
`Pitch.md:15`) and a `repo-doc-tree/eve.json` with dead `.voidbot` links; port it
to `gamecult.thing.surface.v1` or delete it if nothing renders it, and enter the
site in the repo set with target r2 (`thing2:follow_up:site-thing1-surface`; the
component is frozen meanwhile). (2) Brokkr publishes `gamecult.eve.surface.v1` at
two sites in `brokkr-daemon/src/main.rs:362-368`, and no campaign can list it at
the 8-repo bound; rename both ids per table T once a campaign lists Brokkr
(`thing:follow_up:brokkr-wire-unhomed`; no Idunn target runs it, so it waits).

The template (every port cut has the same spec skeleton, so Soul can run the same
falsifications):

1. Fixture first: the consumer's current surface rebuilt as a Thing Script document
   through its language package, loaded into `thing-testkit`, geometry-golden'd.
2. Deletes: its Thing 1 publisher path and every `gamecult.eve.*` reader/writer
   (exact paths from the census, to `file:line`).
3. Commands: each existing command becomes a `CommandDecl` and a handler
   returning an `Outcome` through the package (no state-as-string, M3).
4. Test: the consumer's own suite plus one pipeline smoke that renders the
   consumer's real surface headlessly through core and asserts the geometry golden
   and the command round-trip. Operator check: view it in the TUI/web.
5. Negative grep: no `gamecult.eve` and no `eve-browser-lowering` imports remain.
- **Prior art:** expand/contract (parallel change) and strangler-fig, with the
  caveat in R-K §4 that Thing 2 kinds are new kinds not renames, so an aggregator
  reads two kinds, not one under two ids **[read]**. Confidence high on the
  template, per-consumer specs owe their own census read.

### Piece 14: retirement

Delete Thing 1's tree from the kernel repo after the last port. Test: negative
greps for `gamecult.eve`, `eve-browser-lowering`, `packages/eve-contracts`,
`web/eve-dsl.js`; workspace builds with the directories gone; README front door
(`doctrine`: first screen true) rewritten; Idunn targets (P15) re-pointed. Soul
pass before the tag.

## 4. What every cut's Soul pass attacks (derived, not new)

Rules M1-M10 are the checklist; in addition the shared failure shapes from this
spec: (i) a backend that grows layout, text or focus (compare dependency trees and
public fns); (ii) a decode refusal that reads as absence; (iii) a rule held by a
constructor but skipped by decode (the contract's P18 lesson, now applied to
`CommandInvocation` and results received by core); (iv) a golden re-blessed to
match a bug (Soul reruns generation on a clean checkout and diffs); (v) an `Embed`
or placeholder that silently drops.

## 5. Decisions made here (summary)

| ID | Decision | Confidence |
| --- | --- | --- |
| D1 | One `thing-core` crate with modules; six crates in total | medium-high |
| D2 | Nine-variant closed render-command list | medium |
| D3 | Closed diagnostics enum drives placeholders, logs, a11y and tests | high |
| D4 | Byte-compared binary render and geometry goldens, `bless` task | medium |
| D5 | Invocation lifecycle with core-only `Malformed` and `TimedOut` | medium-high |
| D6 | Cell widths pinned to a `unicode-width` version in goldens | medium |
| D7 | One focus owner in core, id-keyed, modal trap | medium-high |
| D8 | (see O7) edit history scope | open |
| D9 | Web draws through canvas 2D first | medium |

## 6. Open: what this spec could not decide

Each with a recommendation; none blocks cuts 1-8.

- **O1. Parley (and AccessKit types) on `wasm32`.** No source read establishes
  that Parley + fontique build and perform for wasm; GPUI web has its own text
  system and Slint web uses femtovg, not Parley. Recommendation: one half-day
  probe (build `thing-core` text module to wasm, shape `f12`) before cut 9a is
  specced; the fallback is browser `Canvas.measureText` behind the `Measure` trait,
  which weakens "core owns text" for the web only. `thing:ruling:thing2-thing-owns-text`
  already puts shaping in core, so taking the fallback contradicts it and needs a
  new operator ruling. Needs: the probe.
- **O2. Glyph rasterization path** (canvas 2D, Slug, Vello family). Partly ruled:
  rasterization is backend-private and the first GPU backend picks Vello or a Slug
  port by measurement (`thing:ruling:thing2-thing-owns-text`). Still open:
  Recommendation: canvas 2D now; revisit after a measured frame budget (O10).
  Needs: operator word on whether GPU glyphs gate web cutover.
- **O3. The exact command list**, in particular whether `Plot` lowers to
  `Polyline`/`Polygon` in core or becomes an `Embed`. Recommendation: lower in core
  (consumers expect charts; an `Embed` makes every host write one). Needs: review
  of D2 by Meta.
- **O4. CultMesh/CultNet surface for `thing-mesh` and how a browser reaches it**
  (direct WebTransport/QUIC, or a gateway). Not read in this pass. Who owns the
  provider advertisement is ruled (`thing2:ruling:advertisement-owner`: discovery
  leaves Thing, its owner is CultMesh or Odin, follow-up
  `thing2:follow_up:discovery-owner-campaign`); the transport question stays open.
  Needs: a short
  Eyes pass over `cultmesh-ts`/`cultnet-ts` in CultLib before cut 12 and 11c are
  specced.
- **O5. Defaults that are guesses:** invocation timeout 15 s, frame budget, scrollbar
  style, overlay z policy. Recommendation: land as `Config` with these defaults and
  record a follow-up to measure.
- **O6. What only the operator can check:** real IME on Windows/macOS/iOS/Android
  browsers, a real screen reader on the web mirror. No CI substitute exists; the
  spec names them as operator checks at cut 11 and at each consumer cutover.
- **O7. Editing scope (D8):** undo/redo, clipboard, drag-select auto-scroll in 9b.
  Recommendation: undo and clipboard are in (a general library's reasonable
  expectation; clipboard goes through `HostEffects`), word/line navigation in,
  rich text out. Needs: nothing but a go.
- **O8. Locale in `Format`.** Recommendation: a host-supplied `Locale` input,
  default `en`, formats stay a closed set.
- **O9. Diagnostics added by later cuts** (`AssetMissing`, `NoAccessibleName`)
  extend D3's list; confirm the list stays closed and versioned with core.
- **O10. Performance budgets** (frame time at 1k and 10k nodes, wasm size). No
  number was measured anywhere in the sources read; Clay's "microsecond layout"
  claim has no table (R-F §2.1) and the Taffy README numbers are layout-only
  (R-F §2.2). Recommendation: cut 6 records the first measured numbers in a
  bench fixture, and budgets are set from them rather than guessed.
- **O11. Thing Script text syntax: closed by ruling, no open question.**
  `thing2:ruling:thing-script-form` (InForce, ruled 2026-10-10, choice
  `document-authority-text-later`; operator: "all recommended") says the typed
  document is Thing Script's one authority, and a `.thingscript` text syntax comes
  later, parsed only by core. Consequences held in this spec: nothing but the typed
  document is authoritative; no language package or host parses a text form (packages
  build the typed document and nothing else; tooling reaches the one parser through core, which
  the question's recommendation expects to expose via wasm); when the text form is cut
  it lands in `thing-core` next to `resolve` and produces a `SurfaceDocument`, so
  `resolve` still sees only the typed document, and its tests are round-trip
  (text to document to text) plus the existing `resolve` suite on the parsed
  output. It has no cut yet and sits after cut 3; the map's cut order is not
  changed here.
- **O12. Meta's rule.** This spec reads "prior-art sources cited" as: every
  decision names the sources and a confidence, with [read]/[background] honesty.
  If the rule also fixes a format, a minimum source count or a required venue,
  the tags above are where to retrofit it.
- **O13. Where the repo's unmerged contract-ts / contract-cs branches stand.**
  Ruled as far as the mind records: the live specs are `cut-contract-ts.r3` and
  `-cs.r3`, which wait on contract-cs h2 being admitted, held by Soul and merged
  by Self. This spec did not re-verify branch heads; that check belongs to the
  merging Self.

## 7. Substrate gaps felt writing this spec

- Resolved: the `thing2` and `thing` campaigns are now readable
  (`huginn rulings thing2 thing`, `huginn view <id>`), and this spec was checked
  against them on 2026-11-10. Rulings are now cited by id. Remaining gap: the
  `huginn rulings` header line truncates the choice, so each ruling needs a `view`.
- No published golden-command suite exists to copy (D4); the harness is ours to
  build and is itself a cut-1 testkit deliverable.
