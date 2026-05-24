# 0001. Record architecture decisions

Date: 2026-05-23

## Status

Accepted

## Context

This project is partly a working floodlight controller and partly a public worked-example of harness-engineering-style agentic coding. In both roles, decisions are more valuable when they leave a trail. A future contributor (or future agent) reading the code should be able to ask "why is this structured this way?" and find an answer that does not require re-running the original conversation.

Without a lightweight record, decisions tend to (a) get re-debated every few months, (b) get reversed by accident in a refactor, or (c) leave their *why* only in commit messages, which are easy to miss when reading code.

The cost of "ADR everything" is bureaucratic friction; the cost of "ADR nothing" is institutional amnesia. The trade-off is a low bar — ADRs only for decisions whose reasoning survives more than a sentence.

## Decision

We use Architecture Decision Records (ADRs), in the style popularized by Michael Nygard, stored in `docs/adr/`.

- ADRs are numbered sequentially with no gaps (`0001`, `0002`, …).
- File names are `NNNN-kebab-case-slug.md`.
- Each ADR has Status, Context, Decision, Consequences sections.
- Status progresses Proposed → Accepted → Superseded (or Deprecated).
- ADRs are never edited substantively after acceptance. Superseding ADRs replace them.
- `docs/adr/0000-template.md` is the canonical template; copy it when starting a new ADR.

Any decision that survives more than a sentence of reasoning becomes an ADR. Trivia (variable names, formatting) does not.

## Consequences

**Easier:** future contributors can reconstruct intent. Agents reading the repo cold can defer to documented decisions instead of re-deriving them. Reversals are explicit and dated.

**Harder:** every non-trivial PR carries a small extra cost — write the ADR before the code. Reviewers must enforce this, otherwise the convention rots.

**Accepted cost:** some decisions that *feel* small at the time will be written up as ADRs and turn out, in retrospect, not to have needed one. That is fine; the failure mode of under-documenting is worse than the failure mode of mild over-documenting.
