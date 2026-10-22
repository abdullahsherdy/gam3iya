# Gam3iya: Product Requirements Document (PRD)

Version 1.0. Date: 21 October 2026. Status: proposed. Owner: Abdullah Sherdy (mentor, product owner). Builder: Esraa (student developer).
Related: [SRS](SRS.md), [Technical specification](technical-specification.md), [ADR index](adr/README.md), [Technical plan](../plan/technical-plan.md).

## 1. Problem and opportunity

A *gam'iya* is a rotating savings circle common in Egypt. A group of people each contribute a fixed amount every cycle (week or month), and one member receives the whole pot each cycle until everyone has received it once. It runs on trust, and today it is tracked in WhatsApp messages, voice notes, and paper notebooks.

Pain points we have heard repeatedly:

- Nobody has one reliable record of who paid which cycle.
- Disputes about "I paid you on the 5th" cannot be settled.
- The organizer carries the whole burden of chasing late payers and answering "whose turn is it?".
- Payout order is contested when it is decided informally.

**Opportunity.** A transparent, shared ledger that makes the circle easier to run and easier to trust, without handling money.

## 2. Goals and non-goals

### Goals

| # | Goal | How we know |
| --- | --- | --- |
| G1 | An organizer can set up a circle and run it end to end without a notebook | Usability check: a new organizer creates and activates a 10-member circle in under 10 minutes |
| G2 | Every member can see their own status and the full schedule at any time | Member dashboard shows next due date, amount, and payout turn |
| G3 | Every financial change is explainable | 100% of payment and payout changes have an audit entry with actor and time |
| G4 | The app is fast and usable on a phone with an average connection | Lighthouse performance and accessibility 90+ on public pages; p95 API latency under 300 ms on seeded data |
| G5 | The build is a credible portfolio and learning vehicle | Deployed app, CI green, tests, case-study README, system design document |

### Non-goals (version 1)

- **Moving money.** No payment gateway, wallet, card data, or bank integration. The app records what members report and the organizer confirms.
- **Lending, interest, or credit scoring.** Not a financial service.
- **Legal enforcement.** The ledger supports trust; it is not a contract.
- **Native mobile apps.** Mobile-first responsive web only.
- **Offline-first capture.** Treated as a stretch spike, not a requirement.

## 3. Users and personas

| Persona | Description | Needs |
| --- | --- | --- |
| **Mona, Organizer** | 38, accountant, organizes a 12-member monthly circle among colleagues | Set-up in minutes, one place to see who paid, automatic reminders, proof for disputes |
| **Karim, Member** | 26, engineer, member of two circles | Know exactly when and how much to pay, mark payment done, see his payout turn |
| **Platform Admin** | Internal operator (mentor in this program) | Observe health, handle support issues, never edit financial data |

## 4. Scope and release plan

Priorities: **P0** required for the first complete release; **P1** valuable, built if time allows; **P2** future.

| Area | P0 | P1 | P2 |
| --- | --- | --- | --- |
| Account | Register, login, refresh, logout, profile and language | Email verification, password reset | Social login |
| Circle | Create, edit while draft, activate, cancel | Duplicate a circle | Swap-turn requests |
| Members | Invite by link, join, payout order (arranged or random draw) | Remove member in draft | Replacement member mid-circle |
| Payments | Mark paid, confirm, reject with reason, organizer records on behalf | Payment proof upload | Reliability score |
| Payout | Record payout, override with reason | PDF statement export | Admin analytics |
| Visibility | Dashboard, schedule, derived late status, audit trail | Activity feed (NoSQL) | Charts |
| Reminders | In-app notifications | Email reminders via queue | WhatsApp share links |
| Language | Arabic and English, right-to-left layout | Arabic-Indic digits option | More locales |

### Milestone view

| Milestone | Outcome | Weeks |
| --- | --- | --- |
| M0 | Walking skeleton: repo, CI, one page, one test | 1 |
| M1 | Frontend MVP on mock API: typed, tested, deployed | 2 to 4 |
| M2 | Real API and PostgreSQL: auth, circles, payments, audit | 5 to 8 |
| M3 | Next.js, full CI/CD, security and accessibility, production | 9 to 12 |
| M4 | Activity feed, Redis, reminders, scale design, interview readiness | 13 to 16 |

## 5. User stories and acceptance checks

Each check is written so it can become an automated test before the feature is built.

### 5.1 Accounts

- **A1.** As a user I can register with email and password. *Check:* duplicate email is rejected with a clear message; password under 10 characters is rejected; the stored value is never the password.
- **A2.** As a user I stay signed in safely and can sign out everywhere. *Check:* an expired access token is silently renewed; after logout, the old refresh token fails.

### 5.2 Circles and members

- **B1.** As an organizer I create a circle with name, contribution, member count (2 to 30), cycle length, start date, grace days, and payout method. *Check:* contribution must be a positive whole number of piasters; member count outside 2 to 30 is rejected.
- **B2.** As an organizer I invite members with a link. *Check:* the link expires after 7 days; it cannot add more members than the circle has slots; joining after activation is rejected.
- **B3.** As an organizer I choose the payout order by arranging it or by a random draw. *Check:* the draw result is stored with a verifiable commitment and cannot change after activation.
- **B4.** As an organizer I activate the circle when all slots are filled. *Check:* activation with an unfilled slot is rejected; activation generates the full schedule once.

### 5.3 Payments and payouts

- **C1.** As a member I mark my payment for a cycle as paid. *Check:* only my own pending payment can be marked; repeating the request with the same idempotency key does not create a second change.
- **C2.** As an organizer I confirm or reject a marked payment (rejection needs a reason), or record a payment on a member's behalf. *Check:* a member cannot confirm; every outcome writes an audit entry in the same transaction.
- **C3.** As a member I see when I am late. *Check:* lateness is derived from due date plus grace days in Cairo time and is never stored.
- **C4.** As an organizer I record the payout for a cycle once all payments are confirmed. *Check:* payout with unconfirmed payments requires a reason and is flagged in the audit trail; a second payout for the same cycle is rejected.
- **C5.** As an organizer I correct a payment after its cycle closed. *Check:* the correction requires a reason, creates a new version, and keeps the old one visible.

### 5.4 Visibility and reminders

- **D1.** As a member I see my dashboard: next due date, amount, my payout cycle, and circle progress. *Check:* amounts match the schedule examples in Appendix A.
- **D2.** As an organizer I see per-cycle status for every member (paid, marked, pending, late). *Check:* counts on the screen equal the sum from the database.
- **D3.** As a user I receive an in-app reminder two days before a due date and when a payment becomes late. *Check:* reminders are created once per event and can be marked read.

## 6. Experience principles

1. **Mobile first.** Most users open the link from WhatsApp on a phone.
2. **Arabic first-class.** Right-to-left layout is designed in, not patched on.
3. **One obvious next action** on every screen (for example, "Mark as paid").
4. **Trust through transparency.** Everyone in a circle sees the same schedule and statuses.
5. **Never lose money data.** Amounts are exact; actions are safe to retry.

## 7. Success metrics

| Metric | Target |
| --- | --- |
| Time for a new organizer to create and activate a circle | Under 10 minutes |
| Lighthouse performance and accessibility on public pages | 90 or higher |
| API p95 latency on seeded data (10,000 payments) | Under 300 ms |
| Domain package test coverage | 90% or higher |
| API test coverage | 70% or higher |
| Critical findings in the internal security checklist | Zero |
| CI pipeline duration on a pull request | Under 6 minutes |

## 8. Risks and open questions

| Risk or question | Response |
| --- | --- |
| Users may expect the app to move money | State the non-goal clearly on the landing page and in onboarding |
| Handling financial data raises trust and legal questions | Ledger only; no card or bank data; privacy notice; mentor review before any public launch |
| Arabic copy quality | Native-speaker review in week 12 |
| Scope growth | Cut order in the technical plan; P1 only after P0 is green |
| *Open:* maximum realistic circle size and cycle length for target users | Validate with 5 real organizers during week 12 |
| *Open:* should the recipient also pay in their own payout cycle? | **Decided for v1:** yes, every member pays every cycle (see SRS BR-02) |

## Appendix A: Worked examples (these become tests)

**A1. Schedule.** Circle: 10 members, EGP 1,000 per cycle, monthly, start 5 November 2026, grace 3 days. There are 10 cycles. Cycle 1 is due 5 November 2026; cycle 2 on 5 December 2026; cycle 10 on 5 August 2027. Each payout is 10 x 1,000 = EGP 10,000 (1,000,000 piasters).

**A2. Month-end clamp.** Monthly circle starting 31 January 2027: cycle 1 due 31 January, cycle 2 due 28 February 2027 (not a leap year), cycle 3 due 31 March. Due dates are computed from the start date, never chained from the previous due date.

**A3. Lateness.** A payment due 5 November 2026 with 3 grace days is on time through 8 November 2026 and late from 9 November 2026 (Cairo date), if it is not yet confirmed.

**A4. Money.** EGP 1,000.50 is stored as 100050 piasters. Ten such payments total 1,000,500 piasters, displayed as EGP 10,005.00. No floating-point arithmetic is used anywhere.

**A5. Payout with an unconfirmed payment.** Nine of ten payments are confirmed. Recording the payout without a reason is rejected; with a reason, it succeeds and the audit entry carries the reason and a flag.
