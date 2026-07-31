# Contract Formats

A contract is the frozen definition of any surface where one slice touches
another. Contracts are code, not prose. Every contract gets:

- A header: `CONTRACT: <name> — FROZEN <date>` plus **provider** (the slice
  that implements it) and **consumers** (slices that use it).
- The definition itself in the repo's real language/format, committed to main
  at hour zero (usually alongside its stub).

Write contracts in the stack the repo actually uses — match its language,
ORM, and conventions. Formats below are patterns to adapt, not to paste.

## REST endpoints

Define route, method, request/response types, and the error shape. Use the
repo's type system (TypeScript interfaces, Pydantic models, Go structs, JSON
Schema for untyped stacks).

```typescript
// CONTRACT: alerts-api — FROZEN 2026-07-31
// Provider: backend slice. Consumers: dashboard slice.

// GET /api/alerts?since=<ISO8601>&limit=<int, default 50>
interface AlertListResponse {
  alerts: Alert[];
  nextCursor: string | null;
}

interface Alert {
  id: string;            // uuid
  severity: "info" | "warning" | "critical";
  message: string;
  createdAt: string;     // ISO8601
}

// POST /api/alerts/:id/ack  → 204 on success
// Errors (all endpoints): { error: { code: string, message: string } }
// 400 bad input, 401 unauthenticated, 404 unknown id
```

## GraphQL

Contract is SDL plus the named operations consumers will run. Freeze both:
schema changes and new required arguments both break consumers.

```graphql
# CONTRACT: monitor-graph — FROZEN 2026-07-31
type Check {
  id: ID!
  url: String!
  status: CheckStatus!
  lastRunAt: DateTime
}
enum CheckStatus { UP DOWN UNKNOWN }

type Query {
  checks(status: CheckStatus): [Check!]!
}
type Mutation {
  runCheck(id: ID!): Check!
}
```

## Database schema

Contract is the DDL (or migration file) for every table more than one slice
reads. **Migrations have exactly one owner** — one slice writes all
migrations; others request columns through that owner. Two people writing
migrations in parallel is a guaranteed merge-day failure.

```sql
-- CONTRACT: checks-table — FROZEN 2026-07-31
-- Provider: backend slice (owns all migrations). Consumers: monitor slice (read/write status).
CREATE TABLE checks (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  url        TEXT NOT NULL,
  status     TEXT NOT NULL DEFAULT 'unknown',  -- 'up' | 'down' | 'unknown'
  last_run_at TIMESTAMPTZ
);
```

State who may INSERT/UPDATE which columns if write access is split.

## Events / queues

Contract is topic/queue name, payload type, and delivery semantics
(at-least-once vs at-most-once, ordering). Consumers must tolerate the
stated semantics.

```typescript
// CONTRACT: check-events — FROZEN 2026-07-31
// Topic: "check.status_changed" — at-least-once, no ordering guarantee.
// Provider: monitor slice. Consumers: notifier slice.
interface CheckStatusChanged {
  checkId: string;
  from: "up" | "down" | "unknown";
  to: "up" | "down" | "unknown";
  at: string; // ISO8601
}
```

## Environment variables / config keys

Contract is a table committed as `.env.example`. Every key has one owner;
nobody else adds keys without telling the owner (duplicate/renamed keys break
everyone's local runs silently).

```bash
# CONTRACT: env — FROZEN 2026-07-31 (owner: infra slice)
DATABASE_URL=postgres://localhost:5432/app_dev   # infra
API_PORT=3001                                    # backend
VITE_API_BASE=http://localhost:3001              # dashboard
SLACK_WEBHOOK_URL=https://hooks.slack.com/...    # notifier — stub value OK at hour zero
```

## Shared types / modules

A shared module (types package, utils, constants) has **one owner who
writes, everyone imports**. Consumers needing a change file a request to the
owner; they never edit the shared module on their own branch.

```typescript
// CONTRACT: shared/types.ts — FROZEN 2026-07-31 (owner: backend slice)
export type Severity = "info" | "warning" | "critical";
export interface Paginated<T> { items: T[]; nextCursor: string | null; }
```

## Freeze rules and change protocol

1. **FROZEN means frozen.** After hour zero, a contract changes only through
   the protocol below — never by silently editing it.
2. **Change protocol**: proposer announces in the team channel → every
   consumer of that contract agrees → one commit updates contract + stub
   together → announce done. If any consumer objects, the change waits or
   the proposer works around it in their own slice.
3. **Additive beats breaking.** Adding an optional field/endpoint usually
   needs no protocol; renaming, removing, or changing types always does.
4. **Version if contested.** If a breaking change is essential mid-build and
   a consumer can't absorb it, ship `v2` alongside `v1` and delete `v1`
   during integration freeze — don't break a teammate's working slice.
