# Specs

A spec is the unit of planned work in this repo. Every non-trivial change is delivered through one. The root [`CLAUDE.md`](../../CLAUDE.md) is the binding contract; this file is the operational detail.

## Layout

Each spec is a folder under `docs/specs/`:

```
docs/specs/
  2026-05-22-auth-service-bootstrap/
    requirements.md
    design.md
    tasks.md
  2026-06-01-pulse-scanner-worker/
    ...
```

Spec ID: `YYYY-MM-DD-<kebab-slug>`. The date is the day the spec was started (not delivered). The slug is short, specific, and stable — don't rename a spec folder once it has work against it.

## Phases (sequential, gated by user approval)

Each phase corresponds to one file. Each file carries a `status` in its frontmatter (see [Frontmatter](#frontmatter) below) that drives the gate.

1. **Requirements** — create `requirements.md` with `status: draft`. When the content is ready for the user to review, flip to `status: pending-review`. When they approve, flip to `status: approved` and set `approvedAt`. Then move to phase 2.
2. **Design** — create `design.md` with `status: draft`. Same `draft` → `pending-review` → `approved` flow. Then move to phase 3.
3. **Tasks** — create `tasks.md` with `status: draft`. Same flow. Approval of `tasks.md` is the green light for implementation.
4. **Implementation** — work the checklist top-to-bottom, ticking `- [ ]` → `- [x]` as each item lands. Update `updatedAt` on `tasks.md` as you go.
5. **Closeout** — when the last task ticks, set `completedAt` on `tasks.md`, update [`docs/architecture/`](../architecture/), and write an ADR if the spec settled a meaningful alternative.

**Don't skip phases.** A `design.md` written before `requirements.md` is `approved` gets rewritten; a `tasks.md` written before `design.md` is `approved` gets rewritten. The gates exist because the rewrites cost more than the wait.

**Trivial-fix exception:** typos, comment fixes, dependency bumps with no behavioral impact. When in doubt, write a spec.

## Frontmatter

Every spec file starts with YAML frontmatter. Common fields:

```yaml
---
status: draft                       # draft | pending-review | approved
createdAt: 2026-05-22T14:30:00Z     # ISO 8601 UTC — set once, never changed
updatedAt: 2026-05-22T14:30:00Z     # ISO 8601 UTC — bump on every substantive edit
approvedAt: null                    # ISO 8601 UTC — set when the user approves; null until then
---
```

`tasks.md` adds one extra field — the implementation closeout marker:

```yaml
---
status: approved
createdAt: 2026-05-22T14:30:00Z
updatedAt: 2026-06-04T09:15:00Z
approvedAt: 2026-05-23T11:00:00Z
completedAt: 2026-06-04T09:15:00Z   # ISO 8601 UTC — set when the last `- [ ]` ticks; null until then
---
```

Rules:

- **Format:** ISO 8601 in UTC (`YYYY-MM-DDTHH:MM:SSZ`). Always UTC, always with the trailing `Z`, so timestamps sort and compare cleanly regardless of who edited where.
- `status` is the canonical phase signal. The SessionStart hook reads it directly.
- `createdAt` is when you created the file. It does not change.
- `updatedAt` is when you last edited the body. Bump it on every meaningful edit (typo fixes are fine to skip).
- `approvedAt` stays `null` while the file is `draft` or `pending-review`. The moment the user approves, set it to now and flip `status` to `approved`.
- `completedAt` (tasks.md only) stays `null` until every `- [ ]` is ticked. When the last box ticks, set it to now — this is the explicit closeout marker that distinguishes "all boxes ticked, ready to close" from "fully closed".
- Status normally advances `draft` → `pending-review` → `approved`. **An approved file is frozen** — a guard-rail hook blocks content edits while `status: approved`. To revise one, run the loop explicitly: downgrade `status` back to `draft` (or `pending-review`) as a **status-only** edit, make your changes, re-gate approval, then flip `status` back to `approved` as an isolated flip. Approval itself must never ride along with other content changes in the same edit — gate it, then flip status alone.

## File templates

Copy from [`_templates/`](./_templates/):

- [`requirements.md`](./_templates/requirements.md) — what & why
- [`design.md`](./_templates/design.md) — how
- `tasks.md` — the checklist, written from the workflow skill's own template rather than one kept here

## Task checkbox convention

Every actionable item in `tasks.md` is a single markdown checkbox line carrying a leading coord (`1`, `4.1`, `4.1.2`) so it can be ticked individually and addressed by the helper scripts:

```markdown
- [ ] 2.1 Write Hono app entry with /health route
- [x] 2.2 Wire better-auth Drizzle adapter to sql-auth
```

Group with `##` headings (`## Pre-implementation`, `## Implementation`, `## Verification`, `## Documentation`). Avoid nested checkboxes — they hide progress from the SessionStart hook's counter.

## In-progress tracking

There is no list to maintain. The `inject-in-progress-specs.sh` SessionStart hook reads each spec's frontmatter `status` and its `tasks.md` checkbox count straight from disk and injects a summary at session start. Disk is the only source of truth.

## Enforcement: open questions block approval

A PreToolUse hook (`block-approve-with-open-questions.sh`) blocks `Edit` / `Write` calls that would set a `requirements.md` or `design.md` `status` to `approved` while the file still contains an `## Open questions` section. The block message reminds you to resolve the questions with the user and remove the section before retrying. This is a guard rail, not a substitute for actually engaging the user on the open items.

## Enforcement: the approved gate is a clean, one-way flip

A second PreToolUse hook (`block-approved-doc-edits.sh`) guards the `approved` gate on `requirements.md` / `design.md`:

- It blocks an edit that flips `status` to `approved` **and** changes other content in the same operation. Approval is the user's sign-off, not something to smuggle in alongside substantive edits. Land the edits first, get approval, then flip `status` on its own.
- It blocks any content edit **while the file is already `approved`**. The only edit it lets through is a pure status flip: downgrade `approved` → `draft` / `pending-review` to re-open the file, or an idempotent no-op. Changes to `approvedAt` / `updatedAt` ride along with a status flip and don't count as content.

So the revise loop for an approved doc is explicit: **downgrade (status-only) → edit → re-gate approval → flip back to `approved` (status-only)**. `tasks.md` is intentionally exempt — its checklist and `completedAt` are mutated in place all through implementation and closeout.

## When a spec is "done"

A spec is done when `tasks.md` has zero `- [ ]` lines left **and** `completedAt` is set. The moment the last box ticks:

1. Set `completedAt` on `tasks.md` to now (ISO 8601 UTC) and bump `updatedAt`. That alone drops the spec out of the injected in-progress list.
2. Update [`docs/architecture/`](../architecture/) — create a doc if one doesn't exist for the component; otherwise edit.
3. If the spec settled a meaningful architectural choice, write an ADR in [`docs/adr/`](../adr/).

The hook reports `ready to close` for specs whose checklist is fully ticked but whose `completedAt` is still `null` — that's a reminder to perform steps 1-3 above.

The spec folder itself stays — it's the history of how the system got there.
