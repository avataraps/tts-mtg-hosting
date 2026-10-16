# Architecture Decision Records

An ADR captures **one decision** that closed off plausible alternatives, with the context and consequences. ADRs are immutable once accepted — to change a decision, write a new ADR that supersedes the old one.

## When to write one

Write an ADR whenever a design decision:

- Closes off alternatives that a reasonable engineer would otherwise pick (e.g. chose Postgres over Spanner, Confluent Cloud over Aiven, REST over gRPC).
- Constrains future work in a load-bearing way (e.g. "all tenant data is scoped by RLS, not application-layer filtering").
- Is the answer to a "why is it this way?" question we expect to come up more than once.

If a decision is local to one module and easily revisited, prefer notes in the spec's `design.md`. ADRs are for the choices we want to remember.

## File format

- Filename: `NNNN-<kebab-slug>.md`, four-digit sequential ID starting at `0001`.
- Use [`_template.md`](./_template.md).
- Status lifecycle: `Proposed` → `Accepted` → optionally `Superseded by NNNN`. Once `Accepted`, the body does not change.
- To revise: write a new ADR that supersedes the old one, and update the old one's status (the only allowed edit) to `Superseded by NNNN`.

## Enforcement

The `block-adr-edits.sh` PreToolUse hook, shipped with the spec-docs plugin, enforces immutability on `NNNN-*.md` files:

- **A `Proposed` ADR is still a draft** — any edit passes, including body revisions and the `Proposed` → `Accepted` status flip. Enforcement begins at `Accepted`.
- **Append-only edits are allowed** — anything where the `old_string` in the Edit call survives intact inside `new_string`. Use this for adding late-arriving alternatives, links to related ADRs, or follow-up context.
- **Replacing existing text is blocked.** The agent should write a new ADR that supersedes this one instead. One exception: the `Accepted` → `Superseded by NNNN` status flip is allowed when `old_string`/`new_string` are exactly the status line.
- **Bypass for genuine corrections** (typo, broken link, factual error in the original): include the literal marker `[adr-honest-edit]` anywhere in `new_string`. The marker stays in the file as the audit trail. Use sparingly.

The hook does not apply to `README.md` or `_template.md`.

## Index

- [`0001-record-architecture-decisions.md`](./0001-record-architecture-decisions.md) — we use ADRs.
