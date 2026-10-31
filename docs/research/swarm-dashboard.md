# Swarm dashboard: contract state, sources, first cut

Serves Eureka's ruling `swarm-dashboard-first-thing2-surface` (not readable from
the libby mind; cited by the id in the claiming follow-up
`libby-body:follow_up:chat-s354-359`). Read-only pass by the `libby` Self run
`libby:run:mind-20261031T061423908Z`, 2026-10-31. Nothing was built or deployed.
Claims below were probed on 2026-10-31 unless marked **unverified**.

The question the dashboard answers for a stranger: **is anyone working, and on what.**

## 1. Where Thing 2's surface contract stands

Probed against the forge `GameCult/Thing` (main `e119e76`, 2026-10-22) and its
`hands/thing2-*` branches. The source of truth for cut status is the `thing2`
campaign in the Eureka mind, which this pass cannot read; the prose map
(`docs/thing2-map.md`, last edited 2026-10-12) is the evidence used.

| Piece | State |
| --- | --- |
| Contract, Rust (`crates/thing-contract`) | On main. 13-arm closed `Node` union, `SurfaceDocument`, `CompositionDocument`, `CommandDecl/Invocation/Result`, `AssetCatalog`; byte goldens under `tests/goldens/`. |
| Contract, C# and TypeScript | Not on main. Branches `hands/thing2-contract-cs` (`3dadfc6`, 2026-10-14) and `hands/thing2-contract-ts` (`aeede04`, 2026-10-12). The map lists the cross-runtime store check and publication as waiting. |
| Core (resolution, `thing.std` compositions, binding via the CultMesh port, Taffy layout, command dispatch) | Not built. No crate exists. Decoding Verse surfaces waits on `cultlib-gaps:follow_up:decode-stack-safety`. |
| TUI backend | Not built. Planned first backend. |
| Web backend and shell | Not built. |
| Discovery/advertisement | Not a Thing kind (`thing2:ruling:advertisement-owner`); read through a core port; today's `gamecult.eve.provider_advertisement.v1` is unchanged. |

`README.md` says the same: the layout solver, text, accessibility tree, render
commands and backends "are not built yet". **Nothing can render a Thing 2
surface today.** The only live renderers are Thing 1's (browser oracle, Ghostlight's vendored lowering,
the Nightwing browser reference), and Eve dashboards are quarantined (`gamecult.md`: no new Eve surfaces).

### What a provider must publish to render

From `docs/thing2-map.md` "The consumer contract" and `crates/thing-contract/src`:

1. A `SurfaceDocument` (`gamecult.thing.surface.v1`, key `thing:surface:<providerId>.<label>`,
   `version` bumped only on structure change). Closed nodes: `Box, Text, Image, Button, Toggle,
   Slider, TextInput, Select, Repeat, Overlay, Plot, Embed, Use`. A swarm board is `Box` + `Text` +
   `Repeat` (one row per instance) + `Use` of `thing.std` compositions (`card`, `metric`, `key-value`).
2. The **bound state records** the document points at, under the provider's own schema and key. A
   `Bind` is `{provider?, schema, key, member path, format?}` plus `Item` inside a `Repeat`. A
   collection binds to `{provider, schema, key_prefix}` or a list member of one record. No
   expressions, no derived values: **the state's owner publishes derived values** (map rationale,
   P5). So the dashboard's "working / idle" must be a published field, not computed in the surface.
3. Discovery: the provider is found through the existing advertisement, not a Thing record.
4. Commands only if the surface acts. A read-only board declares none.

A provider needs no layout or renderer code. It needs the contract package in its language
(Rust is the only one on main) and typed CultMesh state.

## 2. Sources (all typed state)

### 2a. Run documents, eight minds

Each mind is its own Huginn daemon (`HEARTBEAT_PERSONAS` in `/etc/gamecult/heartbeat/heartbeat.env`):
eureka 17872, libby 17873, heimdall 17874, kiko 17875, aqua 17876, weksa 17877, nibu 17878,
void 17879, all `rudp://10.77.0.1:<port>`. Probed: from this Self run,
`EUREKA_INSTANCE=<name> HUGINN_ENDPOINT=<endpoint> huginn query '{"schemas":["epiphany.pipeline.run.v2"],"limit":200}'`
answers on all eight, read-only, with caller `client unknown` (no per-client read identity yet).
Counts that day: eureka 68, libby 78, heimdall 94, kiko 70, aqua 81, weksa 74, nibu 71, void 87.

`epiphany.pipeline.run.v2` (`huginn schema run`): `instance, label, operated_by (Operator|Mind),
turn (PersonaTurn|SelfRun), host, started_on (a date), budget_usd, claims [PipelineRef], campaigns`.

What the dashboard can and cannot take from it:

- **Open/closed is not on the Run.** It is derived: a Run is closed when a `Resolution` with
  `subject` = the run is in force (`huginn query` prints `InForce` or `Resolved:<outcome>`).
  Resolution key `<instance>:resolution:run.<label>.n<seq>`. `huginn history <runId>` reads it.
- **Start time is a date.** The label `mind-YYYYMMDDTHHMMSSmmmZ` encodes the start to the
  millisecond, but that is a convention in a string. The admission header (`huginn view`) carries
  `admitted <timestamp>`, and the Resolution's header gives the end. Both are header facts, not
  document fields. A dashboard needing start and end times wants them as typed fields.
- **"Outcome" is nearly empty.** Of 78 libby runs, 77 resolved `Recorded` and one is in force.
  The meaning lives in free text: `reason "answered 1"` (55), `"answered 2"` (5), `"the connector
  refused the credential lease: UnknownCaller"` (3), `"unit not active"`, `"failed after 0
  answered..."`, plus rationales `The Persona unit's start ended.` (61) and `The Self run's carrier
  closed the run its claude left open.` (11). Two Self-run resolutions carry the run's whole result
  message inside `reason` (long prose, operator-facing detail). A public board
  must **never project `reason` or `rationale`**.
- **"On what" is only on Self runs.** `claims` holds the follow-up or other document the run took;
  a `PersonaTurn` has no claims (its work is the chat room). The claimed `follow_up.item` is the
  operator's chat-relayed brief, free text, sometimes private. The typed handle (claim id, campaign
  slug, the claimed document's `repo`) is safe; the `item` is not.
- **An open Run can be a dead Run.** Eleven libby Self runs were closed with "carrier closed the
  run its claude left open", so for some interval an in-force Run belonged to nobody. "Open" is
  "admitted and not yet resolved", not "a process is working".
- **Dollar fields:** `budget_usd` is on every Run. Do not bind it.

Snapshot at 06:16 UTC: two Self runs in force, `libby:run:mind-20261031T061423908Z` (this one) and
`void:run:mind-20261031T061543899Z` (claims `void-body:follow_up:chat-s335-340`); no Persona run
in force in any of the eight.

### 2b. Heartbeat state and poll trace

Source: `Huginn/crates/mind-body/src/heartbeat.rs`. `mind-heartbeat` is one Idunn-managed unit, sole
writer of `heartbeat.cc` (a single-file CultCache store under the unit's state directory). Two kinds:

- `eureka.heartbeat.initiative.v1`, key = instance slug: `initiative_usd, as_of, urgency, urgency_at,
  last_burned_run, filled_events, classified_seq, launched_run, launch_spend_usd,
  launch_spend_known, completions_through, launch_window`.
- `eureka.heartbeat.poll_trace.v1`, key = zero-padded unix ms, written only for polls that changed
  something: `at, instances [InstanceTrace], launches [run label], burns [BurnTrace], classifier`.
  `InstanceTrace`: `instance, initiative_usd, rate_usd_per_h, urgency, urgency_age_s, busy,
  released, crossing_at, waited_s, idle_s, filled_by, fill_reason, note`.

Usable without dollars: `busy` (an in-force Persona run, from the mind), `released`, `crossing_at`,
`waited_s`, `idle_s`, `filled_by`/`fill_reason` (`tag` or `name`), `note`, `launches`, `urgency`,
`classifier`. These say **why nobody is working** (waiting for initiative, held by the envelope).
Dollar fields to leave alone: `initiative_usd`, `rate_usd_per_h`, `launch_spend_usd`,
`spend_usd`, and in the Initiative record the same names. The metering teardown removes them, and
`initiative_usd` is the unit initiative itself is denominated in, so the heartbeat's own model will
change under whatever binds this.

Gaps:

- **Not verified live.** `heartbeat.cc` is not readable from this Self run's account, and no
  `/var/lib/gamecult` path for it was visible. The shape above is from source and
  `eureka-body-map.md:534-540`, not from a read of the file.
- **No wire.** The heartbeat writes a file; it does not serve one. A reader opening a single-file
  store while the heartbeat rewrites it has no stated safety (`FileStore::open` pulls the whole file
  per call). A dashboard source must be published state, not a second process reading the file.
  `mind-heartbeat timeline` prints the latest poll for a human.
- The trace is written only when a poll "changed something", so its newest line can be old while
  the system is healthy. Staleness must come from the heartbeat's own published liveness, which
  does not exist as a field.

### 2c. Connector usage readout

Source: `ModelConnector/schemas/usage_readout.json`, `crates/model-connector/src/readout.rs`.
`gamecult.model.usage_readout.v1`: `{schema_id, at_unix_ms, plan_quota: [{window, status,
utilization_permille (nullable), resets_at_unix_ms, observed_at_unix_ms}]}`. Whole-plan windows, no
per-caller field, **no dollars**: the safe source for "how much capacity is left".

Gaps: it is a connector operation (`usage_readout` on `gamecult.model.connector`), reached at the
heartbeat's `MIND_CONNECTOR_ENDPOINT` (`tcp://127.0.0.1:4110`), a loopback port with a per-caller
credential. Three libby Self runs were already refused with `UnknownCaller` on a lease, so a new
caller needs registration before it can ask. It is a request/answer, not a document in a store. A
dashboard needs a publisher that polls it and republishes it as a CultMesh record. `plan_quota`
also rides on every lease grant. **Unverified:** the live values, since asking requires a caller
credential this pass was not given.

### 2d. Not a source: the presence provider

CultLib's 10-29 presence provider attests Idunn-managed **service** liveness, not attention or work
(`libby-body:follow_up:presence-attention-not-carried`). It can answer "is the heartbeat unit up".
It cannot answer "is anyone working".

## 3. Smallest first cut

**Cut 1: one published read model, `swarm.work.v1`, from the Runs of the eight minds. No heartbeat,
no usage, no renderer.**

One record per instance (eight, key `<instance>`), derived and published by a single projector
(owner: the swarm-dashboard provider), holding only closed-set or id fields:

- `state`: `Working` (a Self run in force), `Turn` (a Persona run in force), or `Idle`.
- `run`: label of the newest in-force run, its `turn`, its start (from the admission header), else the
  newest closed run's end.
- `on`: the claim id(s) and campaign slug(s), no free text.
- `observed_at` and the per-mind `as_of`, so a dead projector reads as stale.

This answers the stranger's question with typed state only. It deliberately omits `reason`,
`rationale`, `item`, `budget_usd`, and every `*_usd` field.

The Thing 2 surface for it is a `SurfaceDocument` that is a `Repeat` over those eight records, one
`thing.std.card` each, bound to `state` / `on` / the time fields. It can be authored and golden-tested
with the Rust contract on main. It cannot be seen until core (2a) and a backend exist. That is the
actual gating item, and it is not this cut's to work around with a Thing 1 or Eve surface.

Cut 2: add heartbeat liveness and the why-idle fields (`busy`, `released`, `waited_s`, `idle_s`,
`note`, `launches`), once the heartbeat publishes them as typed CultMesh state (it currently writes a
file only, and the dollar-denominated initiative model is under teardown).
Cut 3: add the `plan_quota` windows once a publisher polls the connector under a registered caller.

### Forks for the operator (recommendations given)

1. **Who is the stranger?** If public, the board shows ids and campaign slugs only (cut 1 as
   written); "on what" in words needs a typed `public_label` that the claiming agent sets, not
   `follow_up.item`. Recommend ids only until that field exists.
2. **Where does the projector live?** Recommend a small reader that queries the eight Huginns (all
   answer today) and publishes `swarm.work.v1`, rather than adding publication to `mind-heartbeat`,
   which is Idunn-managed with a single-file store and a dollar model in flux. Confidence moderate:
   the cost is a new daemon, which needs the independent-lifecycle account `gamecult.md` requires.
3. **Is "open" enough?** Recommend marking `Working` only when a Run is in force **and** its
   carrier is attested (the Run was admitted by a launch that is still live); the data for that
   attestation is not in the Run. Until it is, say in the surface that `Working` means "admitted,
   unresolved".
4. **Typed times.** Recommend asking Huginn for `started_at` and `ended_at` as Run/Resolution
   document fields, so no consumer parses labels or header facts.
5. **Sequencing.** A stranger sees nothing until core and a backend land. If the swarm board should
   come earlier than the TUI backend, say so; the TUI is the planned first backend and gives agents
   a reader, but a stranger needs the web backend (map cut 4).

### Substrate gaps this pass felt

- The `swarm-dashboard-first-thing2-surface` ruling and the `thing2` campaign are in Eureka's mind,
  not readable from libby's: cut status here comes from a 2026-10-12 prose map.
- `heartbeat.cc` unreadable from a Self run, and a file, not a served document.
- No typed `ended_at` / `started_at` on Run; outcome is free text in `reason`.
- No per-client read identity on the eight Huginns (`client unknown`).
