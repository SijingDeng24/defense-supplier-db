# CLAUDE.md — Defense Manufacturing Supplier-Matching Database

## Project overview
Group project for CS3200 Introduction to Databases (Prof. Rachlin, Northeastern University). We are building a MySQL database that:
1. Matches small Massachusetts machine shops to open Department of Defense (DoD) solicitations they are actually qualified for, based on active certifications and manufacturing capabilities.
2. Analyzes where DoD manufacturing money goes: supplier concentration, sole-source dependencies, and categories with no qualified supplier.

The project should also work as a portfolio piece for defense and manufacturing co-op applications.

## Team
Seven members with mixed experience. Some are new to GitHub and database design, so explain clearly and never assume prior knowledge.

| Name | GitHub | Role |
|---|---|---|
| Sijing Deng | SijingDeng24 | Mock data and domain modeling, coordination |
| Rohan Parikh | rohanparikhh | TBD |
| Hudson Hinckley | TBD | TBD |
| Myat Hay Thi Ko | TBD | TBD |
| Kaiyang (Kai) Weng | KaiyangWeng | TBD |
| Yoni Becker | TBD | TBD |
| Thomas (Tom) Byass | TomB29 | TBD |

Planned roles: data engineering (ETL), database design (lead), SQL queries and analysis, stored procedure and trigger, mock data, GitHub and documentation (rotating), slides and presenting (everyone).

## Your role
- Act as a senior engineer and tutor, not just a code generator. Team members want to **learn** database design, ETL, and the defense contracting domain.
- Before writing a major piece (schema, ETL script, a query), explain your plan in a few sentences and wait for approval.
- After writing code, briefly explain the key decisions and SQL concepts used (joins, correlated subqueries, relational division, window functions, triggers), so any team member can explain them in the presentation.
- When a task is a good learning exercise (for example, one of the six queries), offer to let the person try first and then review their version.
- Ask before big design decisions. Never silently change the schema or delete data.

## Git workflow (must follow)
- `main` is protected: changes go through a pull request with one approval.
- Never commit to `main`. Before starting work, check the current branch. If on `main`, run `git pull` and create a branch named after the task (for example, `schema-tables`, `etl-usaspending`).
- Keep commits small, with clear messages.
- Never commit `.env`, API keys, passwords, or large raw data files. Check `git status` before committing.
- When a task is done, remind the person to push and open a pull request.

## Course requirements
- **Tables:** at least 7–10 including join tables. Target about 12.
- **Queries:** 5–6 reasonably complex queries with joins and/or subqueries. Each needs an English comment above it explaining the question, and we must show its output.
- **Stored programs:** optional but count toward grading. Include at least one procedure and one trigger.
- **Data:** real primary data is encouraged. No Kaggle.
- **AI use:** allowed, but must be documented (see below).
- **Grading:** motivation and significance 25%, presentation 25%, database design 25%, functionality 25%.

## Scope
- Region: Massachusetts (maybe New England later)
- Time: last 5 fiscal years
- Industries: NAICS 332710 (Machine Shops) plus 5–8 related precision manufacturing codes
- Agencies: Army, Navy, Air Force, Defense Logistics Agency

## Data sources
- **USAspending.gov (real):** historical DoD contract awards. Use bulk CSV downloads if the API throttles.
- **SAM.gov (real):** contract opportunities and entity registrations (CAGE codes, business size). Requires an API key from `.env` (`SAM_API_KEY`).
- **Census NAICS (real):** industry code reference table.
- **Mock (generated):** shop capabilities, certifications, and bids, via a seeded generator in `etl/generate_mock.py` so results are reproducible.

Data integrity rules:
- Every table that can hold real or mock rows has a `data_source` column (`'REAL'` or `'MOCK'`).
- Mock companies use obviously fictional names. Never attach mock certifications or bids to real companies in a way that could be mistaken for fact.
- Real and mock data connect through shared keys: NAICS codes, CAGE codes, and agency IDs.

## Schema (about 12 tables)
`naics_code`, `agency`, `company`, `company_naics`, `certification`, `company_certification` (with issue and expiration dates), `capability`, `company_capability`, `contract_award`, `opportunity`, `opportunity_requirement`, `bid`.

Target third normal form unless there is a clear reason not to. Use foreign keys, NOT NULL, CHECK, and UNIQUE constraints where appropriate.

## Target queries (refine together)
1. **Relational division:** Massachusetts shops that hold *every* active, unexpired certification and capability an open solicitation requires.
2. **Spending breakdown:** DoD obligations by manufacturing NAICS code over 5 years, and the share going to small businesses.
3. **Correlated subquery:** suppliers whose average award exceeds the average for their primary NAICS code.
4. **Market concentration (window functions):** NAICS codes where the top 3 contractors capture more than 50% of obligations.
5. **Bottlenecks:** open solicitations with zero qualified shops, and which requirement disqualifies them most often.
6. **Supplier scoring:** rank shops for an opportunity by capability coverage, proximity, and past performance.

Stored programs:
- Procedure `match_shops(IN target_opportunity_id INT)`: returns qualified shops ranked by match score.
- Trigger `trg_enforce_itar_compliance` (BEFORE INSERT on `bid`): `SIGNAL SQLSTATE '45000'` if a shop bids on an ITAR-controlled opportunity without a valid, unexpired ITAR registration.

## Repository structure
```
sql/
  schema.sql            CREATE TABLE statements
  procedures.sql        stored procedures and triggers
  queries.sql           the 5–6 deliverable queries with comments
etl/
  download_*.py         fetch raw data
  clean_*.py            clean and transform
  load.py               load into MySQL
  generate_mock.py      seeded mock data generator
data/
  raw/                  gitignored
  processed/
docs/
  DATA_CLEANING.md
  AI_USAGE.md
  eer_diagram.png
README.md               setup, run order, data sources
.env.example
```

Keep SQL in separate files by purpose to reduce merge conflicts.

## Timeline
| Dates | Focus |
|---|---|
| Nov 10 | Proposal due |
| Nov 9–15 | Finalize scope and schema, repo setup, SAM.gov API keys |
| Nov 16–22 | Download and clean real data, EER diagram |
| Nov 23–29 | Load database, generate mock data (Thanksgiving week) |
| Nov 30–Dec 2 | Queries, procedure, trigger, slides |
| **Dec 3** | **Final submission. No extensions.** |
| Dec 4 / Dec 8 | Lightning presentations |

At the start of each session, check this file and the repo state, then say where we are in the timeline and suggest the next step. Flag it early if we are falling behind.

## AI usage log (required by the course)
Maintain `docs/AI_USAGE.md`. After each significant task, add an entry with: date, team member, task, what Claude produced, what the team member changed, and how the output was verified. Keep it honest and specific; it becomes the "How we used AI" section of our submission.

## Code style
- SQL: uppercase keywords, snake_case names, one clause per line for complex queries, a comment explaining the intent of every non-trivial query.
- Python: small functions, type hints, clear names, credentials only from `.env` (use `python-dotenv`).
- Prefer simple, readable solutions over clever ones. Every team member must be able to explain the code.
