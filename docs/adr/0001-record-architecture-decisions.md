# 0001. Record architecture decisions

**Status:** Accepted
**Date:** {{DATE}}
**Deciders:** {{DECIDERS}}

## Context

This repo will accumulate decisions — choices of database, message bus, framework, deployment target, security posture — many of which close off plausible alternatives. Without a durable, append-only record, the reasoning behind each choice evaporates as the team grows and as the original deciders move on.

## Decision

We use **Architecture Decision Records** to capture decisions that close off alternatives. Format and rules are defined in [`README.md`](./README.md). ADRs are stored in this folder, numbered sequentially from `0001`, and are immutable once `Accepted` — superseded, never edited.

## Alternatives considered

- **No formal record; rely on commit messages and Slack** — searchable in theory, ephemeral in practice. New joiners can't find them.
- **A wiki / Notion page** — drifts from the code; hard to require updates as part of code review.
- **Inline in source comments** — fine for a single file's invariants, useless for cross-cutting decisions.

## Consequences

- Every meaningful architectural choice is discoverable in one place, in chronological order.
- The spec workflow's closeout step ("write an ADR if the spec settled a meaningful alternative") has a clear destination.
- ADRs add a small writing tax on decisions. We accept that tax in exchange for the institutional memory.
