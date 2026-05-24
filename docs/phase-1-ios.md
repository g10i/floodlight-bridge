# Phase 1 — iOS app

> **Status:** Not started. This file is a stub. It will be filled out as the first task of Phase 1, before any Swift lands.

## Goal

A SwiftUI app that talks directly to the SOLLA floodlight over BLE Mesh, with no server in the loop.

## MVP scope (locked)

- Power on/off
- Brightness slider (0–100%, via Light Lightness or Generic Level)
- Color temperature slider (~2700K–6500K, via Light CTL Temperature)
- Connection status indicator
- One-time JSON import flow (Files picker → app's documents directory)

## Out of scope

Scenes, schedules, Siri Shortcuts, motion-sensor config, multi-device support, widgets, watchOS.

## Stack

Swift 6 · SwiftUI · Swift Concurrency (`async`/`await`, `actor`) · Observation framework · iOS 26.0 minimum target · single SPM dependency [`nRFMeshProvision`](https://github.com/NordicSemiconductor/IOS-nRF-Mesh-Library).

## Bundle identifier

`com.github.g10i.floodlightbridge`

## Open design questions (to resolve when Phase 1 opens)

- Where to store the imported mesh JSON — `Documents/` (user-visible) vs. `Application Support/` (hidden). Trade-off and decision will become an ADR.
- Generic OnOff Server is not currently bound to an app key on the SollaMaybe node. Phase 1 must do either programmatic binding on first run or include a manual binding step in the README. Decision becomes an ADR.
- Persistent BLE Mesh proxy connection behavior under app backgrounding is unverified. Phase 1 should characterize this empirically and document findings.

## Phase done-criteria

To be enumerated when the phase opens. At minimum: MVP feature list above is exercisable end-to-end against a real device, on a real iPhone, from a clean install.
