# Gam3iya: Software Requirements Specification (SRS)

Version 1.0. Date: 21 October 2026. Status: proposed. Follows the structure of IEEE 830, simplified for a learning project.
Related: [PRD](PRD.md), [Technical specification](technical-specification.md), [ADR index](adr/README.md), [Technical plan](../plan/technical-plan.md).

## 1. Introduction

### 1.1 Purpose

This document states what Gam3iya must do and the qualities it must have, precisely enough to design, build, and test it. Product intent and priorities are in the PRD; how the system is built is in the technical specification.

### 1.2 Scope

Gam3iya is a web application that records and displays the contributions, schedule, and payouts of rotating savings circles. It does **not** move money, hold funds, or integrate with banks or payment providers.

### 1.3 Definitions

| Term | Meaning |
| --- | --- |
| Circle | One rotating savings group with a fixed contribution and member count |
| Cycle | One round of the circle; one member receives the pot per cycle |
| Organizer | The user who created the circle and administers it |
| Member | A user participating in a circle (the organizer is also a member) |
| Pot / payout | Total of all confirmed contributions in a cycle, given to that cycle's recipient |
| Piaster | One hundredth of an Egyptian pound; the unit in which all amounts are stored |
| Grace days | Days after the due date before an unconfirmed payment counts as late |
| Cairo date | Calendar date in the `Africa/Cairo` time zone |

### 1.4 Priorities

P0 must ship; P1 is built if time allows; P2 is future. Requirement identifiers are stable and used by tests and the traceability matrix.

## 2. Overall description

### 2.1 Product perspective

A standalone system: a web client, an HTTP API, a PostgreSQL database, and supporting services (Redis, MongoDB in later phases). No external systems are required for P0.

### 2.2 User classes

| Class | Permissions summary |
| --- | --- |
| Organizer | Everything a member can do in their own circles, plus configure, invite, activate, confirm or reject payments, record payouts, make corrections, view the audit trail |
| Member | View their circles, mark their own payments, view schedule and statuses |
| Platform Admin (P2) | Read-only operational views; no access to edit financial data |

### 2.3 Operating environment

Modern evergreen browsers (last two versions of Chrome, Safari, Edge, Firefox) on phones and desktops; minimum viewport 360 px wide. Server: Node.js LTS, PostgreSQL (current stable major version, 17 or 18).

### 2.4 Constraints

- All money is stored as integer piasters; floating-point money is forbidden.
- All business dates use the `Africa/Cairo` time zone.
- Every financial state change is audited in the same database transaction.
- No personal financial data (cards, bank accounts) is collected.

### 2.5 Assumptions

- Users have an email address and a modern smartphone browser.
- Circle size is between 2 and 30 members.
- Reminders are in-app for P0; email is P1.

## 3. Functional requirements

### 3.1 Accounts and sessions

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-AUTH-01 | The system shall allow registration with unique email, display name, and password of at least 10 characters | P0 |
| FR-AUTH-02 | The system shall store passwords only as salted argon2id hashes | P0 |
| FR-AUTH-03 | The system shall issue a short-lived access token (15 minutes) and a rotating refresh token (7 days) in an httpOnly, Secure, SameSite cookie | P0 |
| FR-AUTH-04 | The system shall detect refresh-token reuse and revoke the whole token family | P0 |
| FR-AUTH-05 | The system shall provide logout that revokes the current token family, and logout-everywhere | P0 |
| FR-AUTH-06 | The system shall let a user choose Arabic or English and persist the choice | P0 |
| FR-AUTH-07 | The system shall support email verification and password reset | P1 |

### 3.2 Circles

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-CIR-01 | A user shall be able to create a circle with name, contribution (positive integer piasters), member count (2 to 30), cycle length (weekly, biweekly, monthly), start date, grace days (0 to 7, default 3), and payout order method | P0 |
| FR-CIR-02 | A circle shall be editable only while in DRAFT | P0 |
| FR-CIR-03 | The system shall activate a circle only when the number of members equals the member count | P0 |
| FR-CIR-04 | On activation the system shall generate all cycles in one transaction and lock the payout order | P0 |
| FR-CIR-05 | The organizer shall be able to cancel a DRAFT or ACTIVE circle with a reason; cancellation is audited | P0 |
| FR-CIR-06 | A circle shall move to COMPLETED when every cycle has a recorded payout | P0 |
| FR-CIR-07 | An organizer shall be able to duplicate a circle | P1 |

### 3.3 Members and invitations

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-MEM-01 | The organizer shall be able to create invitation links that expire after 7 days and cannot exceed remaining slots | P0 |
| FR-MEM-02 | A signed-in user shall be able to join a DRAFT circle through a valid link; the same user shall not join twice | P0 |
| FR-MEM-03 | In ORGANIZER_ARRANGED mode the organizer shall set a unique payout position for each member before activation | P0 |
| FR-MEM-04 | In RANDOM_DRAW mode the system shall assign positions using a cryptographically secure random source and store a verifiable commitment | P0 |
| FR-MEM-05 | The organizer shall be able to remove a member while the circle is DRAFT | P1 |

### 3.4 Payments

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-PAY-01 | The system shall create one payment record per member per cycle on activation, status PENDING | P0 |
| FR-PAY-02 | A member shall be able to mark only their own PENDING payment as MARKED_PAID | P0 |
| FR-PAY-03 | The organizer shall be able to confirm a MARKED_PAID payment, or reject it with a reason, which returns it to PENDING | P0 |
| FR-PAY-04 | The organizer shall be able to record a payment on a member's behalf, directly CONFIRMED, with a note | P0 |
| FR-PAY-05 | Mark, confirm, reject, and record actions shall accept an idempotency key; repeating a request with the same key and payload shall return the original result without a second change | P0 |
| FR-PAY-06 | The system shall derive LATE for any non-confirmed payment whose Cairo date is later than due date plus grace days; LATE shall never be stored | P0 |
| FR-PAY-07 | After a cycle is COMPLETED, payments shall change only through a versioned correction with a mandatory reason; earlier versions remain readable | P0 |
| FR-PAY-08 | A member shall be able to attach a payment proof image | P1 |

### 3.5 Payouts

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-OUT-01 | The organizer shall be able to record one payout per cycle, amount equal to the sum of the cycle's payments | P0 |
| FR-OUT-02 | Recording a payout while any payment is unconfirmed shall require a reason and shall be flagged in the audit trail | P0 |
| FR-OUT-03 | A second payout for the same cycle shall be rejected | P0 |
| FR-OUT-04 | Recording a payout shall set the cycle to COMPLETED | P0 |
| FR-OUT-05 | The system shall export a circle statement as PDF | P1 |

### 3.6 Dashboards, notifications, and audit

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-DASH-01 | A member shall see for each of their circles: next due date and amount, own payout cycle, and progress | P0 |
| FR-DASH-02 | The organizer shall see, per cycle, the status of every member | P0 |
| FR-DASH-03 | Collection endpoints shall use cursor pagination | P0 |
| FR-NOT-01 | The system shall create an in-app notification two days before a due date and when a payment becomes late, once per event | P0 |
| FR-NOT-02 | Users shall be able to list and mark notifications as read | P0 |
| FR-NOT-03 | The system shall send reminder emails through a background queue | P1 |
| FR-NOT-04 | The system shall keep a per-circle activity feed stored in a document database | P1 |
| FR-AUD-01 | The system shall write an append-only audit entry (actor, time, action, entity, before, after, reason, request id) in the same transaction as every financial change | P0 |
| FR-AUD-02 | The application database role shall hold INSERT but not UPDATE or DELETE on the audit table | P0 |
| FR-AUD-03 | The organizer shall be able to view the audit trail of their circle | P0 |

### 3.7 Language and layout

| ID | Requirement | Priority |
| --- | --- | --- |
| FR-I18N-01 | Every user-facing string shall exist in Arabic and English | P0 |
| FR-I18N-02 | The layout shall switch to right-to-left for Arabic | P0 |
| FR-I18N-03 | Dates shall display in the Cairo time zone and amounts as EGP with two decimals | P0 |

## 4. Business rules

| ID | Rule |
| --- | --- |
| BR-01 | A circle has exactly one cycle per member; each member receives the pot exactly once |
| BR-02 | Every member pays in every cycle, including the cycle in which they receive the pot |
| BR-03 | Due date of cycle k is computed from the start date as start + (k - 1) x cycle length; for monthly circles the day of month is clamped to the last day of shorter months |
| BR-04 | Payout amount of a cycle equals contribution x member count |
| BR-05 | Allowed payment transitions: PENDING to MARKED_PAID; MARKED_PAID to CONFIRMED or PENDING (reject); PENDING to CONFIRMED (organizer on behalf); no other transition |
| BR-06 | The payout order cannot change after activation |
| BR-07 | A user sees only circles in which they hold a membership; a request for any other circle behaves as not found |
| BR-08 | Platform administrators cannot create, edit, or delete financial records |

## 5. Non-functional requirements

| ID | Category | Requirement |
| --- | --- | --- |
| NFR-PERF-01 | Performance | p95 latency under 300 ms for read endpoints and under 500 ms for write endpoints on a dataset of 10,000 payments |
| NFR-PERF-02 | Performance | Largest Contentful Paint under 2.5 s on a mid-range phone over a simulated 4G connection for public pages |
| NFR-SEC-01 | Security | Every endpoint enforces authentication and role-and-scope authorization on the server; a role by endpoint test suite proves it |
| NFR-SEC-02 | Security | All input is validated against shared schemas; the API never trusts client-computed amounts or statuses |
| NFR-SEC-03 | Security | Protection against the OWASP Top 10 categories relevant to the stack, checked with an internal checklist and automated scanners |
| NFR-SEC-04 | Security | Secrets are never committed; configuration comes from environment variables; dependencies are scanned |
| NFR-SEC-05 | Security | Login and write endpoints are rate limited |
| NFR-REL-01 | Reliability | Payment and payout operations are atomic; a failure leaves no partial change |
| NFR-REL-02 | Reliability | The database is backed up daily and a restore drill is performed once |
| NFR-A11Y-01 | Accessibility | Meets WCAG 2.2 level AA for all P0 screens; keyboard operable; sufficient contrast |
| NFR-I18N-01 | Localization | Layout is correct in both left-to-right and right-to-left at 360 px width |
| NFR-MNT-01 | Maintainability | Business logic lives in a framework-free domain package with at least 90% test coverage |
| NFR-MNT-02 | Maintainability | Every pull request passes lint, typecheck, tests, and build in CI before merge |
| NFR-OBS-01 | Observability | Structured logs with request id; no passwords, tokens, or full financial payloads in logs |
| NFR-PRV-01 | Privacy | Only email and display name are collected; a user can request deletion, which anonymizes their personal data while preserving the ledger |

## 6. External interfaces

| Interface | Description |
| --- | --- |
| User interface | Responsive web UI, Arabic and English, mobile first |
| HTTP API | JSON over HTTPS, versioned under `/v1`, errors as RFC 7807 problem details, OpenAPI 3.1 document |
| Database | PostgreSQL via Prisma migrations; MongoDB and Redis added in phase 4 |
| Email (P1) | SMTP provider through a queue worker |

## 7. Data requirements

Entities: user, refresh token, circle, membership, invite, cycle, payment, payment version, payout, idempotency receipt, audit event. Retention: financial records and audit events are kept for the life of the circle plus 5 years; refresh tokens and idempotency receipts are purged after expiry. Full schema is in the technical specification.

## 8. Traceability matrix

| PRD story | Requirements |
| --- | --- |
| A1, A2 | FR-AUTH-01 to FR-AUTH-06, NFR-SEC-05 |
| B1 to B4 | FR-CIR-01 to FR-CIR-06, FR-MEM-01 to FR-MEM-04 |
| C1 to C5 | FR-PAY-01 to FR-PAY-07, FR-OUT-01 to FR-OUT-04, FR-AUD-01 to FR-AUD-03 |
| D1 to D3 | FR-DASH-01 to FR-DASH-03, FR-NOT-01, FR-NOT-02 |
| Experience principles | FR-I18N-01 to FR-I18N-03, NFR-A11Y-01, NFR-I18N-01 |
