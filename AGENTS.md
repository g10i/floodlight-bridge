# AGENTS.md

Primary harness file for `floodlight-bridge`. If you are an agent (Claude Code, Codex, Cursor, etc.) or a new human contributor, read this top to bottom before changing anything.

## 1. What is this project?

A reverse-engineered BLE Mesh control plane for a SOLLA outdoor floodlight that lost its vendor app and cloud backend. Ships in three sequential phases: an iOS app (Phase 1), a Mac mini daemon with a LAN HTTP API (Phase 2), and a SmartThings integration (Phase 3). See [README.md](README.md) for the visitor's intro and [seed.md](seed.md) for the original design intent.

## 2. Where do I find context?

- **Intent and rationale:** [seed.md](seed.md) — written before any code, captures locked decisions and known unknowns.
- **Architectural decisions:** [docs/adr/](docs/adr/) — every non-trivial decision lives here as a numbered, dated record.
- **Per-phase design:** [docs/phase-1-ios.md](docs/phase-1-ios.md), [docs/phase-2-daemon.md](docs/phase-2-daemon.md), [docs/phase-3-smartthings.md](docs/phase-3-smartthings.md).
- **BLE Mesh background:** [docs/mesh-primer.md](docs/mesh-primer.md) — short orientation for newcomers to the protocol.
- **Sample data:** [data/sample-mesh-network.json](data/sample-mesh-network.json) — redacted; real exports are gitignored.

## 3. How do I work on it?

Phase 0 has no build step. CI (`.github/workflows/ci.yml`) runs markdown lint and a couple of data-integrity guards on every PR. You can preview locally:

```sh
# markdown lint (requires Node; uses markdownlint-cli2)
npx markdownlint-cli2 "**/*.md"

# sample mesh JSON must parse and have zeroed keys
python3 -c "import json; json.load(open('data/sample-mesh-network.json'))"
```

Build and test commands per phase will be added here as their phase opens.

## 4. What conventions are non-negotiable?

- **One logical change per commit.** If a turn would produce a sprawling diff, split it.
- **ADR or it didn't happen.** Any decision that survives more than a sentence of reasoning becomes an ADR in `docs/adr/`. Numbered sequentially. Status: Proposed → Accepted → Superseded.
- **No silent dependencies.** New Swift packages, shell tools, GitHub Actions — each is called out in the PR description.
- **Never commit a real mesh network export.** The pattern `*mesh*network*.json` is gitignored except for `data/sample-*.json`. CI enforces this.
- **Stop at phase boundaries.** Do not start Phase 1 work while Phase 0 is open; do not start Phase 2 until Phase 1 ships.
- **Honesty over enthusiasm.** When something is uncertain, say so in the PR description. When a chosen path is worse than an alternative, write the ADR comparing them.
- **Prose over bullets in prose contexts; lists where lists belong** (file trees, command lists, status enums, this list).

## 5. What's the current phase?

**Phase 0 — harness materialization.** No application code yet. Flip this line when Phase 1 opens.

## 6. What's out of scope right now?

While Phase 0 is open, do not:

- Create `ios/`, `daemon/`, or `smartthings/` directories.
- Add Swift package manifests, Xcode projects, or any compilable source.
- Touch the Telink vendor model `0x02110000` (deferred indefinitely; not part of any current phase).
- Set up auth, TLS, or remote-access surfaces for the (not-yet-existing) daemon. LAN-only is the v1 plan; anything else is a separate ADR.
- Introduce build/test CI jobs. Those land with the phase that introduces code to build or test.

When Phase 1 opens this list flips to enumerate Phase 1's out-of-scope items (scenes, schedules, Siri Shortcuts, motion-sensor config, multi-device, widgets, watchOS).
