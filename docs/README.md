# docs/

Three kinds of documentation live here. Pick the right one for what you're writing.

| Folder | Kind | Time-bound? | When to write |
|---|---|---|---|
| [`architecture/`](./architecture/) | Evergreen architecture docs | No — kept in sync with the code as the system evolves | After (or alongside) the spec that introduced the system component. |
| [`adr/`](./adr/) | Architecture Decision Records | No — but **immutable** once accepted; supersede with a new ADR instead of editing | Whenever a decision closes off a plausible alternative. See [`adr/README.md`](./adr/README.md). |

The spec workflow is the entry point for every change. ADRs and architecture docs are the durable artifacts that outlive any individual spec.
