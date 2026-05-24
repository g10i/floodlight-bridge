# BLE Mesh primer

A very short orientation for newcomers. This is not a substitute for the [Bluetooth Mesh Profile 1.0.1 spec](https://www.bluetooth.com/specifications/specs/mesh-profile/), but it is enough to read this repo's code and design docs without immediately drowning.

## What BLE Mesh is

Bluetooth Mesh is a managed flood network built on top of BLE advertising. Nodes relay messages to each other, so range is the sum of individual hops rather than a single node-to-controller link. It's a SIG (standards-body) protocol — *not* the same as proprietary BLE protocols that vendors sometimes call "mesh" in marketing.

A network is identified by a **net key**. Within a network, **app keys** scope which applications can use which models on which nodes. Both kinds of keys are 128-bit. Keys never leave the network — this repo's redacted sample at [data/sample-mesh-network.json](../data/sample-mesh-network.json) replaces them with zeros and CI verifies this.

## Nodes, elements, models

A **node** is a single device. Each node has one or more **elements** (think: independently addressable sub-units of the same physical device — a dimmer with two channels has two elements). Each element exposes one or more **models** — small, well-defined behaviors with their own opcodes.

The SOLLA floodlight in this project has two elements:

- **Element 0** exposes Configuration, Health, Generic OnOff, Generic Level, Generic Default Transition Time, Generic Power OnOff (+ Setup), Light Lightness (+ Setup), Light CTL (+ Setup), Scene (+ Setup), Scheduler (+ Setup), Time (+ Setup), and a Telink vendor model (`0x02110000`) of unverified purpose.
- **Element 1** exposes Generic Level and Light CTL Temperature.

For Phase 1, the relevant models are **Generic OnOff** (on/off), **Light Lightness** (brightness), and **Light CTL Temperature** on Element 1 (color temperature).

## Model addresses

- `0x1000` — Generic OnOff Server
- `0x1002` — Generic Level Server
- `0x1300` — Light Lightness Server
- `0x1303` — Light CTL Server
- `0x1306` — Light CTL Temperature Server

A full list is in the [SIG-assigned-numbers model spec](https://www.bluetooth.com/specifications/assigned-numbers/).

## Provisioning, in one paragraph

A new mesh node enters the network through **provisioning** — a key exchange that hands the node its net key and gives it a unicast address. SOLLA's floodlight supports insecure (no-OOB) provisioning, which is how the [Nordic nRF Mesh app](https://apps.apple.com/app/nrf-mesh/id1380726771) was able to add it to the author's network. After provisioning, an **app key** is bound to each model the controller wants to talk to. *Provisioning is not the same as binding* — a model can be present, provisioned, and still untalkable-to until an app key is bound to it. (This is exactly the state Generic OnOff is in on the SollaMaybe node right now; see Phase 1's open questions.)

## Why this matters for the project

The hardware is a standards-compliant mesh node. We do not need vendor servers, vendor apps, or vendor protocols. Everything we want — on/off, brightness, color temperature, eventually scenes and schedules — is a standard model with documented opcodes. The Phase 1 app talks to it through [`nRFMeshProvision`](https://github.com/NordicSemiconductor/IOS-nRF-Mesh-Library), which handles the network layer, encryption, and proxy connections for us.
