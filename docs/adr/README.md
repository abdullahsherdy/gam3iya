# Architecture decision records (ADR index)

Each decision has a status, the alternatives considered, and a **validation and reversal** note: the evidence that confirms it, and what would make us reverse it. Esraa writes the full ADR file for each decision in the week shown, using the template at the end.

| ADR | Decision | Status | Validated in | Alternatives considered |
| --- | --- | --- | --- | --- |
| 001 | pnpm monorepo with shared `contracts` and `domain` packages | Accepted | Week 1 | Separate repositories; copy types by hand |
| 002 | Prisma as ORM | Proposed, spike in week 6 | Week 6 | Drizzle; raw SQL with a query builder |
| 003 | PostgreSQL is the system of record; money as integer piasters | Accepted | Week 5 | MongoDB for everything; decimal columns |
| 004 | Modular monolith, not microservices | Accepted | Week 6 | Microservices; serverless functions |
| 005 | Authentication: hand-built JWT with rotating refresh tokens first, compared with a library | Proposed | Week 7 | Better Auth; managed identity provider |
| 006 | MongoDB for the activity feed and notifications (learning-driven polyglot persistence) | Proposed | Week 13 | PostgreSQL JSONB table (the production default) |
| 007 | Outbox pattern between PostgreSQL commits and MongoDB writes | Proposed | Week 13 | Dual write; change data capture |
| 008 | Hosting and delivery: GitHub Actions with preview, staging, and approved production | Proposed | Week 11 | Platform-only deploy hooks |
| 009 | Migrate the web app from Vite to Next.js App Router in phase 3 | Accepted | Week 9 | Stay on Vite and add a separate marketing site |
| 010 | Reminders through a BullMQ queue and worker | Proposed | Week 14 | Cron job in the API process |

## Validation and reversal notes

- **ADR-002:** run the same five tests with both ORMs (scoped create, duplicate command, version conflict, row lock, audit grants). Reverse if the scoped-repository pattern cannot be enforced cleanly.
- **ADR-005:** reverse to a library if the hand-built flow cannot pass the refresh-reuse test or exceeds the budget by more than 3 hours.
- **ADR-006:** the honest production default is PostgreSQL JSONB. MongoDB is kept because the learning goal includes document modeling. Reverse to JSONB if the outbox cost exceeds 6 hours or the feed adds no learning value.
- **ADR-009:** reverse if the Next.js migration of the dashboard is not complete by the end of week 10; keep the public pages in Next.js and the dashboard in Vite.

## Template

```markdown
# ADR-NNN: Title
## Status
Proposed | Accepted | Reversed | Superseded by ADR-NNN
## Context
What problem and constraints led to this decision?
## Decision
What did we choose?
## Alternatives considered
- Option: why not
## Consequences
- Positive:
- Negative:
## Validation and reversal
What evidence confirms this? What would make us reverse it?
```
