# 0002. Three-phase sequential architecture

Date: 2026-05-23

## Status

Accepted

## Context

The eventual product surface is broad: a native iOS app, an always-on Mac mini daemon with a LAN HTTP API, and a SmartThings integration that unlocks voice assistants, geofencing, and routine automation. Each surface is independently useful but they share a single small domain model — the device, its mesh node, and the set of Bluetooth SIG models it exposes (OnOff, Level, Light Lightness, Light CTL).

The shape of the work could be approached three ways:

1. **Parallel** — build all three surfaces concurrently, sharing a Swift package for the domain layer from day one.
2. **Big-bang** — design the full architecture (including the daemon's API and the SmartThings integration) before any code lands, then build top-down.
3. **Sequential, narrow phases** — ship one surface at a time, in dependency order, letting each phase teach the next.

Parallel work requires more design certainty than the author has at the outset. The protocol behavior, the actor model around `nRFMeshProvision`, and the realities of iOS BLE backgrounding are all empirical questions that Phase 1 is best positioned to answer cheaply. Committing to a shared package shape *before* Phase 1 has shipped risks designing for assumptions that don't survive contact with the hardware.

Big-bang carries the same problem in a worse form and adds calendar drag — nothing useful exists until all of it does.

Sequential narrow phases trade away apparent efficiency (some Phase 1 code will be refactored when Phase 2 starts) for two things: each phase produces a useful deliverable, and each phase's lessons are available before the next phase begins.

The cost we are accepting is that Phase 1 and Phase 2 will be developed against an unknown future shared interface, and some Phase 1 code will need to be lifted into a shared layer when Phase 2 opens. Based on the model exposed by the device (standard SIG models, well-defined opcodes), 80%+ of the domain code should transfer cleanly.

## Decision

The project ships in three sequential phases, with hard phase boundaries:

- **Phase 1 — `ios/`** — Native SwiftUI app, BLE Mesh control direct to device. MVP: on/off, brightness, color temperature, connection status, JSON import.
- **Phase 2 — `daemon/`** — Swift command-line tool launched by `launchd` on a Mac mini, maintaining a persistent BLE Mesh proxy connection, exposing a small LAN HTTP API. Reuses the `MeshController` actor and domain model from Phase 1.
- **Phase 3 — `smartthings/`** — Virtual switch / dimmer / color-temp devices, webhooked to the Phase 2 daemon via Routines. Exact integration mechanism (cloud virtual device vs. Edge driver in Lua) is deferred to a Phase 3 ADR.

Hard rules:

- No phase begins while a prior phase is open.
- Phase-specific directories (`ios/`, `daemon/`, `smartthings/`) do not exist in the repo until their phase opens.
- Decisions made in one phase that constrain a later phase get an ADR. Decisions deferred to a later phase are listed explicitly in that phase's design doc.

## Consequences

**Easier:** every phase is shippable on its own. Each phase's empirical lessons (especially Phase 1's, around BLE backgrounding and `nRFMeshProvision` behavior) inform the next phase's design. The repo is always in a coherent state — there are no dead `ios/` directories during Phase 2.

**Harder:** some refactoring is inevitable when domain code moves from Phase 1 into a shared layer for Phase 2. Phase 3 contributors cannot start work until Phase 2 ships. There is no opportunity for parallel contributor effort across phases.

**Accepted cost:** the refactor at the Phase 1 → Phase 2 boundary. This is a known cost and the right place to pay it — refactoring a small, working iOS app's domain layer is much cheaper than designing the shared layer upfront against assumptions that may not survive Phase 1.

See also: [ADR 0003](0003-agents-md-as-primary-harness.md) on the AGENTS.md convention; the per-phase design docs at [docs/phase-1-ios.md](../phase-1-ios.md), [docs/phase-2-daemon.md](../phase-2-daemon.md), [docs/phase-3-smartthings.md](../phase-3-smartthings.md).
