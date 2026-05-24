# Phase 3 — SmartThings integration

> **Status:** Not started. This file is a stub. It will be filled out when Phase 2 ships and Phase 3 opens.

## Goal

Three virtual devices (switch, dimmer, color-temp) in the user's SmartThings account, with Routines that webhook the Phase 2 daemon. Unlocks Alexa, Google Home, geofencing, and "good night" routines for free.

## Open design questions (to resolve when Phase 3 opens)

- Implementation path: virtual-device-via-SmartThings-cloud-API vs. SmartThings Edge driver written in Lua. Trade-offs in latency, hub dependency, account portability, and code-language fit (Lua isn't great, but Edge drivers are local and don't depend on cloud reachability).
- Authentication / pairing model between SmartThings and the Phase 2 daemon (currently no-auth on the LAN — that may need to change here).
- State sync: SmartThings expects to be authoritative for device state; the daemon currently is. Reconciliation strategy.

## Phase done-criteria

To be enumerated when the phase opens.
