# Phase 2 — Mac mini daemon

> **Status:** Not started. This file is a stub. It will be filled out when Phase 1 ships and Phase 2 opens.

## Goal

A Swift command-line tool, launched by `launchd`, that maintains a persistent BLE Mesh proxy connection to the floodlight and exposes a small HTTP API on the LAN. Reuses the `MeshController` actor and domain model from Phase 1; the difference is the surface (HTTP handlers instead of SwiftUI views) and the lifecycle (long-running daemon instead of foreground app).

## API shape (provisional)

```http
POST /floodlight/on
POST /floodlight/off
POST /floodlight/level     { "value": 0..100 }
POST /floodlight/ctl       { "kelvin": 2700..6500 }
GET  /floodlight/status
```

LAN-only, no auth in v1. Anything beyond LAN is a separate ADR.

## Host

macOS 26.3 (Tahoe) Mac mini, always-on.

## Open design questions (to resolve when Phase 2 opens)

- HTTP framework choice (Vapor, Hummingbird, plain `Network.framework`).
- Daemon lifecycle: full `launchd` `KeepAlive` vs. `RunAtLoad` only.
- Connection-loss recovery: backoff strategy, when to surface to clients vs. when to silently reconnect.
- BLE range from Mac mini to floodlight is unverified. If dropouts are common, an ESP32 BLE proxy node may need to be introduced as Phase 2.5.

## Phase done-criteria

To be enumerated when the phase opens.
