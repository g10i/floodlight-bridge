# floodlight-bridge — Project Seed

> This document is the seed for an agentic coding project. The first task for any agent reading this is to **propose a Phase 0 plan to materialize the harness described below — without writing application code yet.** Subsequent phases follow only after the harness exists, is in version control, and CI is green.

---

## 1. What this project is

A reverse-engineered control plane for a SOLLA outdoor floodlight whose vendor app and cloud backend are no longer functional. The light is a standards-compliant Bluetooth Mesh node (Telink TLSR-series chip), already provisioned into a private mesh network owned by the project author.

The project exists for three reasons, in priority order:

1. **Recover capability** — restore on/off, dimming, and color-temperature control to the device, and add modern surfaces (phone app, voice, schedules) that the dead vendor app never offered well.
2. **Serve as a public testing ground for harness-engineering-style agentic coding** — every artifact, convention, and workflow choice should be defensible as a deliberate harness decision, not an accident. The repo doubles as a worked example others can fork.
3. **Be educational for the author** — concretely teach BLE Mesh, Swift concurrency, SwiftUI, and the integration shape between a phone app, a daemon on a Mac, and a SmartThings hub.

If a decision tradeoff arises, prefer (2) over (1) when the cost is small. The point of this repo is *how it was built* as much as *what it does*.

## 2. Hardware and protocol facts (do not re-derive these)

- **Device:** SOLLA outdoor security floodlight (model unknown; ordered via Amazon link `https://a.co/d/00FDNZ2P` — link may rot).
- **Chip:** Telink Semiconductor (Company ID `0x0211`), almost certainly TLSR8258 family.
- **Protocol:** Standard Bluetooth SIG Mesh, profile 1.0.1. Insecure provisioning (no OOB). The device also advertises a vendor GATT service `0xFFFF` with characteristics `FF53` / `FF54` — this is a separate code path the SOLLA app may have used and is **not** required for control. All control flows through the standard mesh stack.
- **Provisioning state:** The author's iPhone, running Nordic's nRF Mesh app, has already provisioned the device into a network named "nRF Mesh Network" with a node named "SollaMaybe" at unicast address `0x0003`.
- **Models exposed (per node composition data):**
  - **Element 0 (Primary, location `0x0001`):** Configuration Server, Health Server/Client, Generic OnOff Server (`0x1000`), Generic Level Server (`0x1002`), Generic Default Transition Time Server (`0x1004`), Generic Power OnOff Server/Setup (`0x1006`/`0x1007`), Light Lightness Server (`0x1300`), Light Lightness Setup Server (`0x1301`), Light CTL Server (`0x1303`), Light CTL Setup Server (`0x1304`), Scene Server (`0x1203`), Scene Setup Server (`0x1204`), Scheduler Server (`0x1206`), Scheduler Setup Server (`0x1207`), Time Server (`0x1200`), Time Setup Server (`0x1201`), and a Telink vendor model `0x02110000` (purpose unverified — likely SOLLA-specific motion-sensor configuration; **out of scope** for v1).
  - **Element 1 (Secondary, location `0x0000`):** Generic Level Server (`0x1002`), Light CTL Temperature Server (`0x1306`).
- **Currently bound to app key:** Generic Level Server on both elements (which is why dimming and color-temp control already work via nRF Mesh). **Generic OnOff Server is not yet bound** — Phase 1 must include either (a) a manual binding step in the README, or (b) programmatic binding on first run, before on/off works cleanly.
- **Mesh JSON export:** A redacted sample of the exported network is committed at `data/sample-mesh-network.json`. The author's real export is **not** committed and must be supplied at runtime.

## 3. Roadmap (three phases, sequential)

The project ships in three loosely-coupled phases. Each phase produces something useful on its own; later phases reuse 80%+ of the prior phase's domain modeling.

### Phase 1 — `ios/` — Native iOS app
**Goal:** A SwiftUI app that talks directly to the floodlight over BLE Mesh, with no server in the loop.

**MVP scope (locked):**
- Power on/off
- Brightness slider (0–100%, via Light Lightness or Generic Level)
- Color temperature slider (~2700K–6500K, via Light CTL Temperature)
- Connection status indicator
- One-time JSON import flow (Files picker → app's documents directory)

**Out of scope for Phase 1:** scenes, schedules, Siri Shortcuts, motion-sensor config, multi-device support, widgets, watchOS.

**Stack:** Swift 6, SwiftUI, Swift Concurrency (`async`/`await`, `actor`), Observation framework. Single dependency: [`nRFMeshProvision`](https://github.com/NordicSemiconductor/IOS-nRF-Mesh-Library) via Swift Package Manager.

### Phase 2 — `daemon/` — Always-on Mac mini bridge
**Goal:** A Swift command-line tool, launched by `launchd`, that maintains a persistent BLE Mesh proxy connection to the light and exposes a small HTTP API on the LAN. Reuses the `MeshController` actor and domain model from Phase 1; the difference is the surface (HTTP handlers instead of SwiftUI views) and the lifecycle (long-running daemon instead of foreground app).

**API shape (provisional, subject to revision in Phase 2 design doc):**
```
POST /floodlight/on
POST /floodlight/off
POST /floodlight/level     { "value": 0..100 }
POST /floodlight/ctl       { "kelvin": 2700..6500 }
GET  /floodlight/status
```
LAN-only, no auth in v1. If exposed beyond LAN, that's a separate ADR.

### Phase 3 — `smartthings/` — SmartThings integration
**Goal:** Three virtual devices (switch, dimmer, color-temp) in the user's SmartThings account, with Routines that webhook the Phase 2 daemon. Unlocks Alexa/Google Home, geofencing, and "good night" routines for free.

Decision deferred to Phase 3 design doc: virtual-device-via-cloud-API vs. SmartThings Edge driver in Lua.

## 4. Tech and convention decisions (locked)

| Decision | Choice | Rationale |
|---|---|---|
| Repo name | `floodlight-bridge` | Descriptive, brand-neutral, ages well if hardware changes. |
| iOS bundle ID | `com.github.g10i.floodlightbridge` | Reverse-DNS style tied to GitHub username. Bundle IDs disallow hyphens. |
| Harness primary | `AGENTS.md` | OpenAI-popularized convention; vendor-neutral; Claude Code reads it natively. |
| Harness alias | `CLAUDE.md` | One line: "This project uses AGENTS.md. See AGENTS.md." |
| License | MIT | Simplest; permissive; compatible with `nRFMeshProvision` (BSD-3). |
| Minimum iOS | 26.0 | Modern SwiftUI/Observation; no back-compat shims; matches author's devices. |
| Mac mini host | macOS 26.3 (Tahoe) | Author's existing always-on machine. |
| Mesh JSON | Redacted sample committed; real one gitignored | Educational; safe; users supply their own at runtime. |
| Phase 0 | Explicit, before any Swift | Project is also a harness-engineering deliverable; clean before/after. |

## 5. Phase 0 — Harness materialization (do this first)

**No application code yet.** Phase 0 produces the scaffolding that every later phase plugs into. Concretely:

### 5.1 Repository layout
```
floodlight-bridge/
├── AGENTS.md                  # primary harness entry point
├── CLAUDE.md                  # alias pointer to AGENTS.md
├── README.md                  # human-facing project intro
├── LICENSE                    # MIT
├── .gitignore                 # ignores user mesh JSON, Xcode junk, .DS_Store, build/
├── .editorconfig              # consistent whitespace across editors
├── docs/
│   ├── adr/
│   │   ├── 0000-template.md
│   │   ├── 0001-record-architecture-decisions.md
│   │   └── 0002-three-phase-architecture.md
│   ├── phase-1-ios.md
│   ├── phase-2-daemon.md
│   ├── phase-3-smartthings.md
│   └── mesh-primer.md         # short BLE Mesh background for newcomers
├── data/
│   └── sample-mesh-network.json   # redacted; keys zeroed
├── .github/
│   ├── workflows/
│   │   └── ci.yml             # lint + (later) build
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md
│       └── feature_request.md
└── .claude/                   # Claude Code-specific (skills, commands) — populated as needed
```

Phase-specific subdirectories (`ios/`, `daemon/`, `smartthings/`) are **not** created in Phase 0. They appear when their phase begins.

### 5.2 `AGENTS.md` contents (target shape)
The file should answer, for an agent or a new contributor, in this order:
1. **What is this project?** (two sentences, link to README for more)
2. **Where do I find context?** (pointers into `docs/`)
3. **How do I work on it?** (build, test, lint commands per phase; for Phase 0, just `./scripts/check.sh` if anything)
4. **What conventions are non-negotiable?** (commit style, ADRs for non-trivial decisions, one PR per logical change, no committing real mesh JSON)
5. **What's the current phase?** (a single line — flip when phase advances)
6. **What's out of scope right now?** (an explicit list — keep agents from drifting into Phase 2 work during Phase 1)

### 5.3 ADR workflow
Every non-trivial architectural decision (e.g. "we use `nRFMeshProvision` directly rather than wrapping it," "we store the mesh JSON in `Application Support` not `Documents`") gets a short markdown ADR in `docs/adr/`. Numbered sequentially. Status: Proposed → Accepted → Superseded. The first two ADRs (`0001`, `0002`) are seeded as part of Phase 0.

### 5.4 CI
A minimal GitHub Actions workflow that runs on every PR. In Phase 0 it does only:
- Markdown lint on docs
- Verifies `data/sample-mesh-network.json` parses as JSON and contains zeroed keys (regex check on hex strings)
- Verifies no file matching `*mesh*network*.json` outside `data/sample-*.json` is committed

Build/test steps are added when their phase introduces compilable code.

### 5.5 README
Human-readable intro: what it is, why it exists, current status (which phase), how to try it, link to `AGENTS.md` for contributors and agents. No marketing voice — this is a hobby project, written like one.

### 5.6 Phase 0 done-criteria
- Repo pushed to GitHub at `g10i/floodlight-bridge`, public, MIT-licensed.
- All files in §5.1 present and minimally non-empty.
- CI green on `main`.
- An ADR exists for the three-phase architecture decision and another for the AGENTS.md+CLAUDE.md convention.
- Running `tree -L 2` on the repo matches the layout in §5.1.

## 6. Working agreement with the agent

- **Plan before doing.** First response to this seed should be a Phase 0 execution plan (file list, ADRs to write, CI scope), not file creation. The author approves before generation begins.
- **One logical change per commit.** If a single agent turn would produce a sprawling diff, split it.
- **ADR or it didn't happen.** Any decision that survives more than a sentence of reasoning becomes an ADR.
- **No silent dependencies.** New SPM packages, new shell tools, new GitHub Actions — each is called out in the PR description.
- **Stop at phase boundaries.** Do not start Phase 1 work while Phase 0 is open. Do not start Phase 2 until Phase 1 ships.
- **Style: prose over bullets in prose contexts; lists where lists belong** (e.g. file trees, ADR statuses, this very list).
- **Honesty over enthusiasm.** When something is uncertain, say so in the PR description. When a chosen path is worse than an alternative, write the ADR comparing them.

## 7. Known unknowns (call out, don't paper over)

- The Telink vendor model (`0x02110000`) almost certainly hides motion-sensor configuration. We don't know its opcodes. Phase 1 ignores it; a future phase may sniff and document it.
- BLE range from the Mac mini to the floodlight is unverified. If Phase 2 hits dropouts, an ESP32 BLE proxy node may need to enter the architecture as Phase 2.5.
- The exported mesh JSON's `configComplete: false` flag on the SollaMaybe node suggests not all models have been queried for Subscription/Publish state. Phase 1 should complete configuration on first run.
- iOS-side persistent BLE Mesh proxy connection behavior under app backgrounding is not fully understood by the author and should be characterized empirically during Phase 1.

---

## Provenance

This seed document was drafted on **May 3, 2026** by **Claude Opus 4.7** (Anthropic), in conversation with **Gil Shklarski**, who directed the work.

The conversation began with a dead vendor app: Gil's SOLLA outdoor floodlight had been orphaned when its iOS app was pulled from the App Store and its cloud backend stopped responding. Over the course of the session, we worked through the diagnostic chain — establishing that the device was Bluetooth-based, then ruling out Tuya/Magic Home/LEDnet ecosystems, then tentatively concluding it spoke a proprietary GATT protocol over service `0xFFFF`, then revising that conclusion when Nordic's nRF Mesh app successfully provisioned it as a standards-compliant SIG Mesh node. Element-by-element model inspection confirmed full OnOff / Level / Light Lightness / Light CTL / Scene / Scheduler support. Brightness and color-temperature control were verified working through nRF Mesh before any project planning began.

From that working baseline, Gil decided to make the recovery effort his first public GitHub project and a deliberate testing ground for harness-engineering-style agentic coding (per OpenAI's published practice). The architectural decisions captured in this seed — three sequential phases, AGENTS.md as primary harness file, Phase 0 before any application code, redacted mesh JSON committed for educational value, MIT license, iOS 26 minimum target, `com.github.g10i.floodlightbridge` bundle identifier — were each elicited from Gil one question at a time and locked before drafting began.

The seed is intentionally agent-readable: §6 (Working Agreement) and the lead instruction at the top set the expectation that the first agent reading this file produces a Phase 0 plan, not Phase 0 code.

*Project status as of seed: pre-Phase-0.*

