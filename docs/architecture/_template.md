# <Component or topic name>

One-paragraph summary: what this is, who depends on it, and where it lives in the codebase.

## Overview

The shape at a glance. Boundaries, scope, what this doc does and does not cover.

## Components

The internal structure — modules, packages, services, or layers that make up this piece of the system. A table works well when there are more than a few.

| Component | Path | Responsibility |
|---|---|---|
| ... | ... | ... |

## Interfaces

What this exposes and what it consumes. API surfaces, message topics, database schemas, environment contracts. Be explicit about names, ownership, and stability guarantees.

## Data model

Persistent state owned by this component — tables, collections, queues, blob layouts. Skip if the component is stateless.

## Operational notes

How it's deployed, monitored, and recovered. Where logs and metrics live. Known operational pitfalls.

## Related

- Specs that introduced or changed this: ...
- ADRs constraining this: ...
- Adjacent architecture docs: ...
