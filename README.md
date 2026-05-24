# floodlight-bridge

A reverse-engineered control plane for a SOLLA outdoor floodlight whose vendor app and cloud backend stopped working. The device is a standards-compliant Bluetooth Mesh node (Telink TLSR-series chip); this project rebuilds control around it from open standards.

## Status

**Pre-Phase-1.** Phase 0 (harness scaffolding) is in progress. No application code yet — that begins in Phase 1.

## Why this repo exists

The light came with an iOS app that got pulled from the App Store and a cloud backend that no longer answers. The hardware is fine; only its software ecosystem rotted. Phase 1 is a SwiftUI app that talks BLE Mesh directly to the device — no vendor servers in the loop.

It is also a deliberate testing ground for harness-engineering-style agentic coding. Every convention here — `AGENTS.md` as the primary entry point, ADRs for non-trivial decisions, an explicit Phase 0 before any Swift is written — is a harness choice, not an accident. The repo is meant to be forkable as a worked example as much as it is meant to control a light.

## Roadmap

The project ships in three sequential phases. Each is useful on its own; later phases reuse the domain model from earlier ones.

- **Phase 1 — `ios/`** — Native SwiftUI app, on/off + brightness + color temperature, talking BLE Mesh directly. See [docs/phase-1-ios.md](docs/phase-1-ios.md).
- **Phase 2 — `daemon/`** — Always-on Mac mini bridge with a small LAN HTTP API. See [docs/phase-2-daemon.md](docs/phase-2-daemon.md).
- **Phase 3 — `smartthings/`** — Virtual devices in SmartThings so Routines, Alexa, and Google Home can drive the light. See [docs/phase-3-smartthings.md](docs/phase-3-smartthings.md).

## How to try it

Nothing to run yet. When Phase 1 lands, the iOS app will require an iPhone on iOS 26+, the [Nordic nRF Mesh app](https://apps.apple.com/app/nrf-mesh/id1380726771) to provision your light, and the exported mesh network JSON imported into the app.

## For agents and contributors

Start with [AGENTS.md](AGENTS.md). It points to the current phase, the working agreement, and the conventions that are non-negotiable. Architectural decisions live in [docs/adr/](docs/adr/). The seed document that originated the project is at [seed.md](seed.md) — useful for understanding intent, less useful for understanding current state.

## License

MIT. See [LICENSE](LICENSE).
