# 0003. AGENTS.md as primary harness, CLAUDE.md as alias

Date: 2026-05-23

## Status

Accepted

## Context

This repo is meant to be agent-readable as a first-class concern, not as an afterthought. Different agentic coding tools have converged on different conventions for where to put their entry-point instructions:

- **OpenAI's Codex** popularized `AGENTS.md` — vendor-neutral, increasingly adopted across tooling.
- **Anthropic's Claude Code** historically reads `CLAUDE.md` but also reads `AGENTS.md` natively.
- **Cursor** reads `.cursorrules` or `.cursor/rules/`, but also respects `AGENTS.md`.
- **GitHub Copilot Workspace** reads `.github/copilot-instructions.md`.

The choice is between (a) maintain one canonical file under a vendor-neutral name and use tiny pointer files for vendor-specific entry points, or (b) duplicate substantial content across vendor-specific files.

Duplication rots fast. The first time an instruction changes in one file but not the others, the project's documented conventions and its actual conventions silently diverge — and which file the next agent reads becomes a coin flip.

A pointer-based scheme keeps the content in exactly one place. The cost is one extra hop for any agent that hardcodes a vendor-specific filename; the benefit is a single source of truth.

`AGENTS.md` is the strongest candidate for the canonical name because it is the most vendor-neutral and is recognized by the broadest set of tools. `CLAUDE.md` is the only vendor-specific pointer worth maintaining today — Claude Code is the project's primary agent — but the same pattern extends if other vendor pointers are needed later.

## Decision

- `AGENTS.md` is the primary harness file. All harness content — current phase, conventions, build commands, out-of-scope list, pointers into `docs/` — lives there.
- `CLAUDE.md` exists only as a one-line pointer: "This project uses AGENTS.md. See AGENTS.md."
- If support for another agent tool ever requires its own filename (e.g. `.cursorrules`), it follows the same pattern: a one-line pointer to `AGENTS.md`. No duplicated content.
- `.github/copilot-instructions.md` is not created speculatively; add it (as a pointer) only if and when Copilot Workspace is actually used on the repo.

## Consequences

**Easier:** one place to update conventions. Agents that find `AGENTS.md` directly use it. Agents that look for `CLAUDE.md` first follow the one-line pointer with negligible overhead.

**Harder:** an agent that *only* reads `CLAUDE.md` and refuses to follow a pointer would miss the harness content. No current agent does this, but the failure mode exists in principle.

**Accepted cost:** the project visibly favors OpenAI's naming convention even though Claude Code is its primary agent. This is deliberate — vendor-neutrality wins over vendor-affinity in long-lived repo conventions.
