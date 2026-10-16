# Architecture docs

Evergreen documentation of the system as it currently exists. Updated alongside the code that implements each component.

## What lives here

One markdown file per major system or topic. Pick names that read well in a directory listing: a future reader should be able to scan filenames and know which doc covers what. Examples:

- `<component>.md` — a deployable or major module (its boundaries, dependencies, schema layout)
- `<integration>.md` — an external system or contract (message bus topics, third-party API surface)
- `<cross-cutting-concern>.md` — patterns that span the system (auth, tenancy, observability)

_None exist yet — they appear via the specs that introduce each system._

Use [`_template.md`](./_template.md) as the starting point for a new architecture doc.

## How it differs from specs and ADRs

- A **spec** ([`docs/specs/`](../specs/)) captures the plan for a single change. Once the change lands, the spec is history; the code is the source of truth.
- An **ADR** ([`docs/adr/`](../adr/)) captures the *why* behind a decision that closed off alternatives. Immutable.
- An **architecture doc** (this folder) captures the *what* — the system as it stands today. It is rewritten as the system evolves.

When code changes invalidate an architecture doc, updating the doc is part of the spec that made the change. The doc's git history is the freshness signal.
