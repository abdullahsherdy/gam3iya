# Gam3iya: project documentation pack

A rotating savings circle (gam'iya) manager, built over 16 weeks as the flagship project of a full-stack mentorship. This pack is the single source of truth for what is built, why, and in what order.

## Contents

| Path | What it is | Read it |
| --- | --- | --- |
| [docs/PRD.md](docs/PRD.md) | Problem, users, scope, user stories with acceptance checks, worked examples (Appendix A) | Week 1 |
| [docs/SRS.md](docs/SRS.md) | Numbered functional and non-functional requirements, business rules, traceability | Week 1 |
| [docs/technical-specification.md](docs/technical-specification.md) | Architecture, database, API, domain logic, security, testing, delivery, scale path | Weeks 5 to 8, 15 |
| [docs/adr/README.md](docs/adr/README.md) | Architecture decisions with alternatives and reversal rules | Write each ADR in the week shown |
| [plan/technical-plan.md](plan/technical-plan.md) | Hours, schedule, checkpoint, cut order, milestones and work packages, risks | Every week |
| [plan/technical-plan.yml](plan/technical-plan.yml) | Machine-readable twin of the plan | Tooling |
| [study-guides/](study-guides/) | One study guide per week (16 Word files) | Sent before each week |

## How to start (week 1)

1. Create the repository and copy `docs/` and `plan/` into it.
2. Read the PRD and SRS; write down every question.
3. Create the calibration log (`plan/calibration-log.csv` with columns: work package, type, estimate hours, actual hours, AI used, note).
4. Follow the Week 01 study guide.

## Rules that never change

Tests first from the PRD acceptance checks. Every change is a pull request. Money is integer piasters. Every financial change is audited in the same transaction. The mentor reviews authentication, authorization, payment, and audit code before merge.
