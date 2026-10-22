# Technical plan: Gam3iya, 16-week mentorship build

Date: 21 October 2026. Status: proposed. Sources: [PRD v1.0](../docs/PRD.md), [SRS v1.0](../docs/SRS.md), [technical specification](../docs/technical-specification.md), [ADR index](../docs/adr/README.md). Machine-readable twin: [technical-plan.yml](technical-plan.yml).

## 1. Commitment and how it is measured

| Item | Value |
| --- | --- |
| Scope | Full PRD P0. Selected P1 (email reminders, activity feed, PDF export) in milestone 4 only if on schedule. P2 excluded. |
| Team | 1 student developer (Esraa) and 1 mentor reviewer (Abdullah); AI assistant under the policy in section 7 |
| Duration | 16 weeks |
| Capacity | 8 independent hours per week = 128 hours, plus 2 live sessions of 90 minutes per week (teaching and review, not counted as build time) |
| **Planned work** | **112 hours** across 41 work packages |
| **Reserved buffer** | **16 hours** (12.5%), allocated by week in section 2 |
| Load factor | 112 / 128 = 87.5% |

The capacity is an assumption to confirm with Esraa in week 1. The estimates are targets, not promises: the week-2 checkpoint (section 3) replaces them with measurement, and the cut order (section 4) protects the deadline.

## 2. Schedule

| Week | Milestone | Work packages | Planned h | Buffer h | Gate |
| --- | --- | --- | --- | --- | --- |
| 1 | M0 | 0.1, 0.2, 0.3, 0.4 | 7 | 1 | M0 exit |
| 2 | M1 | 1.1, 1.2 | 7 | 1 | **Week-2 checkpoint** |
| 3 | M1 | 1.3, 1.4, 1.5 | 8 | 0 |  |
| 4 | M1 | 1.5, 1.6, 1.7, 1.8 | 7 | 1 | M1 exit and mentor demo |
| 5 | M2 | 2.1, 2.2 | 7 | 1 |  |
| 6 | M2 | 2.3, 2.4 | 8 | 0 |  |
| 7 | M2 | 2.5, 2.6 | 7 | 1 |  |
| 8 | M2 | 2.7, 2.8, 2.9 | 6 | 2 | M2 exit and mentor demo |
| 9 | M3 | 3.1, 3.2 | 7 | 1 |  |
| 10 | M3 | 3.3, 3.4, 3.5 | 8 | 0 |  |
| 11 | M3 | 3.6, 3.7 | 6 | 2 |  |
| 12 | M3 | 3.8, 3.9, 3.10 | 7 | 1 | M3 exit; users test the app |
| 13 | M4 | 4.1, 4.2 | 6 | 2 |  |
| 14 | M4 | 4.3, 4.4 | 7 | 1 |  |
| 15 | M4 | 4.5, 4.6, 4.7 | 8 | 0 | System design review |
| 16 | M4 | 4.8, 4.9, 4.10 | 6 | 2 | M4 exit and demo day |

Milestone totals:

| # | Milestone | Weeks | Planned h |
| --- | --- | --- | --- |
| M0 | Walking skeleton and setup | 1 | 7 |
| M1 | Frontend MVP on a mock API | 2-4 | 22 |
| M2 | Real API and PostgreSQL | 5-8 | 28 |
| M3 | Next.js, CI/CD, and production quality | 9-12 | 28 |
| M4 | Document store, Redis, scale, interviews | 13-16 | 27 |
| | **Total** | | **112** |

## 3. Week-2 checkpoint

At the end of week 2, compute the measured ratio **actual hours / estimated hours** per work type (domain, ui, api, database, devops, testing, docs) from the calibration log (WP 0.4).

- **At or below 1.25 overall:** continue as planned.
- **Above 1.25:** apply cuts from section 4 in order until the remaining estimate fits the remaining capacity. Never cut tests, scope enforcement, audit, money integrity, or the pull request workflow to recover time.
- Record the result and any cuts in this file. Estimating honestly is itself a skill she is being taught, so a miss is data, not failure.

## 4. Pre-agreed cut order

1. PDF statement export and payment proof upload (P1).
2. Email reminders; keep in-app reminders.
3. MongoDB activity feed falls back to a PostgreSQL JSONB table; document modeling is learned in a one-day spike instead.
4. Playwright reduced to one main flow.
5. BullMQ worker replaced by a scheduled job in the API process.

## 5. Practices adopted from the veterinary-app plan

The veterinary-app plan targets a two-developer team and a much larger domain. We keep its engineering discipline and leave its domain-specific machinery behind.

| Adopted practice | Replaces | Where it lands |
| --- | --- | --- |
| Walking skeleton first (M0) and spikes tied to ADR "validation and reversal" notes | Building features before the architecture is proven | M0, ADR index |
| Shared TypeScript packages for contracts and business calculations | Duplicated types and logic in web and API | ADR-001, `packages/` |
| Command-style writes with idempotency receipts and payload hash | Naive POST handlers that double-apply on retry | Spec section 6.1, WP 2.7 |
| Append-only audit written in the same transaction, with database grants | Application-level logging only | Spec 3.4, WP 2.8 |
| Scoped repositories and "direct API forbidden" tests | Ad hoc permission checks per endpoint | Spec section 7, WP 2.6 |
| Calibration log, week-2 checkpoint, pre-agreed cut order | Untracked estimates and surprise deadline misses | Sections 3 and 4 |
| Test-first from acceptance checks and worked examples | Tests written after the fact | PRD Appendix A, WP 1.1 |
| AI workflow rules with CI-enforced checks and human review of sensitive code | Unreviewed generated code | Section 7 |
| Documented decisions with sign-offs | Silent assumptions | Section 11 |

**Deliberately not adopted:** offline-first sync (PowerSync), embedded BI dashboards (QuickSight), a React Native mobile app, Oracle and external-system integration, and a generated admin console. These are the right tools for that product and the wrong complexity for this learning path. An admin console and an offline spike remain optional stretch topics.

## 6. Settled decisions

### Platform
- TypeScript strict everywhere; pnpm monorepo; PostgreSQL 17 or 18 as the system of record; all money as integer piasters; all business dates in `Africa/Cairo`.
- Weeks 1 to 4 run against an MSW mock API; the real API replaces it in week 6.
- Next.js replaces Vite in week 9 (ADR-009); the migration is incremental, public pages first.

### Domain
- Every member pays every cycle, including their payout cycle (BR-02).
- Lateness is derived, never stored (FR-PAY-06). Due dates are computed from the start date (BR-03).
- A payout with unconfirmed payments needs a reason and is flagged (FR-OUT-02).
- After a cycle is completed, only versioned corrections with a reason are allowed (FR-PAY-07).

### Authentication
- Hand-built in week 7 to learn the mechanics, then compared with a library in ADR-005.

### Persistence
- MongoDB and Redis arrive in milestone 4 as deliberate learning choices; PostgreSQL stays the system of record (ADR-006, ADR-007).

## 7. AI assistant policy and workflow

The goal is a strong engineer, not a finished repository. Used badly, an AI assistant produces code she cannot explain; used well, it speeds her up.

- **Weeks 1 to 4:** tutor only. She may ask for explanations, reviews, and hints, but writes the code herself.
- **Weeks 5 to 16:** pair allowed for boilerplate, test scaffolding, and refactors. She writes the domain logic, authentication, payment, and audit code herself.
- **Always:** explain-back before merge (she explains every change in her pull request in her own words); tests first from the PRD acceptance checks; the mentor reviews all authentication, authorization, payment, and audit code before merge.
- **CI enforces:** no unscoped repository access, no floating-point money (lint rule), audit written in the same transaction (integration test), no secrets in the repository.
- **Calibration log per work package:** estimate (set before starting), actual hours, work type, whether AI was used.

## 8. Milestones and work packages

Hours are planned build hours. Each milestone has an exit gate.

### M0: Walking skeleton and setup (weeks 1, 7 h)

Exit gate:

- Repository on GitHub with protected main and a PR template
- CI green on a pull request (lint, typecheck, test)
- ADR-001, ADR-003, ADR-009 drafted
- Diagnostic task reviewed; calibration log has its first entries

| WP | Week | Work | Hours |
| --- | --- | --- | --- |
| 0.1 | 1 | Monorepo: pnpm workspaces, strict TypeScript, ESLint, Prettier, Vite app, empty domain and contracts packages | 3 |
| 0.2 | 1 | Git workflow: branch protection, PR template, issue board, conventional commits, first CI workflow | 2 |
| 0.3 | 1 | Read PRD and SRS; draft ADR-001, 003, 009; diagnostic task | 1 |
| 0.4 | 1 | Calibration log and AI-assist policy file | 1 |

### M1: Frontend MVP on a mock API (weeks 2-4, 22 h)

Exit gate:

- PRD Appendix A examples A1 to A5 pass as automated tests
- Domain package coverage 90% or higher
- UI complete for stories B1, B4, C1 to C3, D1, D2 against the mock API
- CI green; preview deployed; mentor demo done

| WP | Week | Work | Hours |
| --- | --- | --- | --- |
| 1.1 | 2 | Domain package: money, schedule, payment status, payout rule, tests from PRD Appendix A written first | 5 |
| 1.2 | 2 | Contracts: Zod schemas, inferred types, typed seed data, parse at boundaries | 2 |
| 1.3 | 3 | App shell: routing, layout, design tokens, Arabic and English, right-to-left | 3 |
| 1.4 | 3 | Circles list and create-circle form with validation | 3 |
| 1.5 | 3-4 | Circle detail: members, cycle table, derived status badges | 3 |
| 1.6 | 4 | TanStack Query with MSW mock API; loading, error, empty states; optimistic mark-paid | 3 |
| 1.7 | 4 | Component tests and domain coverage gate | 2 |
| 1.8 | 4 | Full CI pipeline and preview deployment; M1 demo | 1 |

### M2: Real API and PostgreSQL (weeks 5-8, 28 h)

Exit gate:

- All P0 endpoints implemented; OpenAPI lints clean
- Role by endpoint authorization matrix at 100% of endpoints
- Idempotency, audit-grant, and refresh-reuse tests pass
- API coverage 70% or higher; frontend wired to the real API; deployed on managed PostgreSQL

| WP | Week | Work | Hours |
| --- | --- | --- | --- |
| 2.1 | 5 | ERD, Prisma schema, migrations, constraints, seed, Docker Compose PostgreSQL | 5 |
| 2.2 | 5 | Fifteen reporting queries in SQL (late payers, totals per circle, next payout) | 2 |
| 2.3 | 6 | NestJS skeleton: config, health, validation, RFC 7807 filter, OpenAPI; ORM spike for ADR-002 | 3 |
| 2.4 | 6 | Circles, members, invites, cycles endpoints; activation transaction | 5 |
| 2.5 | 7 | Authentication: argon2id, JWT, rotating refresh tokens with reuse detection | 4 |
| 2.6 | 7 | Scoped repositories and role by endpoint authorization test matrix | 3 |
| 2.7 | 8 | Payment state machine, transaction, idempotency receipts, payouts | 4 |
| 2.8 | 8 | Append-only audit and database role grants | 1 |
| 2.9 | 8 | Indexes with EXPLAIN ANALYZE, cursor pagination, deploy to managed PostgreSQL; M2 demo | 1 |

### M3: Next.js, CI/CD, and production quality (weeks 9-12, 28 h)

Exit gate:

- Next.js app in production behind an approved release pipeline
- Lighthouse 90 or higher on public pages; zero critical items on the security checklist
- Playwright main flow green in CI
- Case-study README and all ADRs marked Accepted or Reversed with evidence

| WP | Week | Work | Hours |
| --- | --- | --- | --- |
| 3.1 | 9 | Next.js app: landing page and public circle page with metadata and SEO | 4 |
| 3.2 | 9 | Port layouts and dashboard; server versus client component boundaries | 3 |
| 3.3 | 10 | Cookie session and middleware; protected routes | 3 |
| 3.4 | 10 | Server actions for create circle, mark paid, confirm; revalidation | 3 |
| 3.5 | 10 | Caching rules, loading and error boundaries | 2 |
| 3.6 | 11 | CI/CD: matrix, caching, previews, protected main, approvals, release tags, Dependabot | 4 |
| 3.7 | 11 | Docker images, environment and secrets management, backup and restore drill | 2 |
| 3.8 | 12 | Security checklist, accessibility audit, Core Web Vitals pass | 3 |
| 3.9 | 12 | Playwright end-to-end main flow | 3 |
| 3.10 | 12 | Case-study README and ADR completion | 1 |

### M4: Document store, Redis, scale, interviews (weeks 13-16, 27 h)

Exit gate:

- Activity feed and notifications in MongoDB through the outbox; Redis rate limiting; reminder worker running
- k6 report meets NFR-PERF-01
- System design document for 1 million users reviewed by the mentor
- Three mock interviews completed with at least Level 3 in 6 of 8 rubric areas

| WP | Week | Work | Hours |
| --- | --- | --- | --- |
| 4.1 | 13 | MongoDB activity feed and notifications, outbox publisher, read API | 4 |
| 4.2 | 13 | Database landscape decision guide (key-value, graph, time-series, columnar, vector) | 2 |
| 4.3 | 14 | Redis: rate limiting and idempotency cache | 3 |
| 4.4 | 14 | BullMQ reminder worker and in-app reminders | 4 |
| 4.5 | 15 | k6 load test, p95 baseline, index review | 3 |
| 4.6 | 15 | System design document: scaling to 1 million users | 3 |
| 4.7 | 15 | Four practice system designs with the mentor as interviewer | 2 |
| 4.8 | 16 | Polish: UX, accessibility, Arabic review fixes, README | 2 |
| 4.9 | 16 | CV, LinkedIn, and GitHub profile README | 2 |
| 4.10 | 16 | Three mock interviews and demo day | 2 |

## 9. Inputs and due dates

| Input | Owner | Due |
| --- | --- | --- |
| Laptop, GitHub account, hosting accounts | Student | Week 1 |
| Diagnostic task completed | Student | Week 1 |
| Managed PostgreSQL and hosting free-tier accounts | Student | Week 4 |
| Native Arabic reviewer | Mentor | Week 11 |
| Five real organizers to test the app | Mentor | Week 11 |
| Two engineers for mock interviews | Mentor | Week 15 |
| Pull request review turnaround of 24 hours | Mentor | Every week |

A late input moves the dependent feature to a labelled placeholder; it does not move the deadline.

## 10. Risks

| Risk | Effect | Response |
| --- | --- | --- |
| Measured ratio above 1.25 | Deadline missed | Week-2 checkpoint and section 4 cut order |
| Hand-built authentication overruns | Week 7 slips | ADR-005 reversal rule: switch to a library if over budget by 3 hours |
| Next.js migration friction | Phase 3 slips | ADR-009 reversal rule: public pages only |
| AI over-reliance | Skills not built | Policy in section 7; explain-back; oral questions in each session |
| Free-tier limits change | Hosting disruption | Decision recorded in ADR-008; Docker keeps the app portable |
| Inconsistent weekly hours | Cumulative delay | Weekly buffer; mentor checks progress at each session |
| Scope creep | Core quality drops | P1 starts only after the milestone gate passes |
| Single mentor | Review bottleneck | 24-hour review turnaround; written checklists reduce review time |

## 11. Decisions the mentor must confirm

1. Gam3iya is the single project; the earlier interview-tracker idea is dropped and its learning goals (MongoDB, Redis, queues) are absorbed into milestone 4.
2. The product is a ledger only; no public launch with real users without a privacy and legal review.
3. Authentication is built by hand first, then compared with a library.
4. The AI assistant policy in section 7.
5. The 8 independent hours per week capacity assumption.
6. MongoDB is kept as a deliberate learning choice even though PostgreSQL JSONB is the production default.
7. The veterinary-app plan stays confidential and separate; only its practices are reused here.
