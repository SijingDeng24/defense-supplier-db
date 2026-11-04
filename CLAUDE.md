# CLAUDE.md — Defense Manufacturing Supplier-Matching Database

## Project overview
This is a group class project for CS3200 Introduction to Databases (Prof. Rachlin, Northeastern University). We are building a MySQL database that:
1. Matches small U.S. manufacturers (machine shops) to open Department of Defense (DoD) contract opportunities they are actually qualified for, based on certifications and manufacturing capabilities.
2. Analyzes where DoD manufacturing money goes and where the supplier base is thin (supplier concentration and capability gaps).

The project should also work as a portfolio piece for defense and manufacturing co-op applications, and as a prototype for a possible business idea.

Team: Sijing Deng + [teammate names]. Group number: [XX].

## Your role
- Act as a senior engineer and tutor, not just a code generator. Team members want to **learn** database design, ETL, and the defense contracting domain.
- Before writing a major piece (schema, ETL script, a query), explain your plan in a few sentences and wait for approval.
- After writing code, briefly explain the key decisions and any SQL concepts used (joins, correlated subqueries, relational division, window functions, triggers), so we can explain them in our presentation.
- When a task is a good learning exercise (for example, writing one of the six queries), offer to let us try first and then review our version.
- Ask before making big design decisions. Don't silently change the schema or delete data.

## Course requirements (must meet)
- **Tables:** 7–10 minimum including join tables. Target is about 12.
- **Queries:** 5–6 reasonably complex queries using joins and/or subqueries. Each must have an English comment above it explaining the question it answers, and we must show its output.
- **Stored programs:** stored procedures, functions, and triggers are optional but count toward assessment. Include at least one procedure and one trigger.
- **Data:** real primary data is encouraged. **No Kaggle datasets.** Mock data (including AI-generated) is allowed.
- **AI use:** allowed, but we must clearly explain how AI was used. See "AI usage log" below.
- **Deliverables:**
  1. `groupXX_queries_and_output.pdf` — queries with comments and output (not zipped)
  2. `groupXX_presentation.pdf` — 5–6 slides, PDF not .pptx; one slide must be the EER diagram
  3. Supporting code (zip)
  4. Database backup (zip), if manageable
- **Rubric:** Motivation & Significance 25%, Presentation 25%, Database Design 25%, Functionality & Capability 25%.

## Tech stack
- MySQL 8 (database), MySQL Workbench (EER diagram, backups)
- Python 3 with pandas and requests (data download and ETL), mysql-connector-python or SQLAlchemy (loading)
- Optional UI: Streamlit or a small Flask/Express app, only if time allows after core deliverables are done
- Git/GitHub for version control; keep API keys in a `.env` file that is listed in `.gitignore`

## Data sources
**Real (public):**
- **USAspending.gov** — federal contract awards. Filter to DoD awarding agencies and manufacturing NAICS codes (start with 332710 Machine Shops and related 332/336 codes). Limit scope (for example, Massachusetts and New England, last 5 fiscal years) so the data stays manageable.
- **SAM.gov** — contract opportunities (solicitations) and public entity registration data (CAGE codes, NAICS, business size). Requires a free API key.
- **Census NAICS codes** — industry code lookup table.

Always check the current official documentation for API endpoints, parameters, rate limits, and bulk-download formats. Do not rely on memory for these; they change. If a source is unavailable or a field doesn't exist, tell us instead of guessing.

**Mock (AI-generated):**
- Shop capabilities (machines, materials, tolerances, finishing processes), certifications held (with expiration dates), and bids.
- Make mock data realistic: small shops with 1–5 certifications, common materials (6061 aluminum, 1018 steel, 304 stainless, Delrin), and realistic capabilities (3-axis and 5-axis milling, CNC turning, waterjet, anodizing, inspection/CMM).
- Include deliberate edge cases: expired certifications, shops that meet some but not all requirements, solicitations no shop can fulfill.
- Use a fixed random seed so data is reproducible.

## Data integrity rules
- **Never present mock data as real.** Every table or row that is synthetic must be traceable as mock (for example, a `data_source` column with values like `USASPENDING`, `SAM`, `CENSUS`, `MOCK`). The presentation and README must state which data is real and which is mock.
- Do not invent statistics, company facts, or findings. Claims in the presentation must come from our query output or a cited source.
- Mock companies must use fictional names. Do not attach fake bids, certifications, or failures to real companies.
- Log data cleaning decisions (dropped rows, deduplication, how missing values were handled) in `docs/DATA_CLEANING.md`, since messy real data is part of the story.

## Safety and compliance rules
- Use **only public data**. No controlled unclassified information (CUI), no export-controlled technical data, no real technical drawings.
- "ITAR-controlled" jobs in the mock data are just a boolean flag on a fake opportunity. Never generate realistic technical details for weapons or defense articles.
- Do not scrape sites that prohibit it; use official APIs and bulk downloads.

## Draft schema (to be refined together)
- `naics_code` (real)
- `agency` (real: Army, Navy, Air Force, DLA, etc.)
- `company` (real from SAM entity data for awards analysis; mock for matching shops; flagged by `data_source`)
- `company_naics` (join)
- `certification` (AS9100, ISO 9001, ITAR registration, CMMC level, etc.)
- `company_certification` (join, with `issued_date`, `expires_date`)
- `capability` (process/machine types)
- `company_capability` (join, with attributes such as max part size or tolerance)
- `contract_award` (real from USAspending)
- `opportunity` (real from SAM.gov, plus mock `itar_controlled` flag if needed)
- `opportunity_requirement` (required certifications and capabilities; may be split into two join tables)
- `bid` (mock)

Use proper primary keys, foreign keys, NOT NULL and CHECK constraints, and indexes on join columns. Target third normal form unless there's a clear reason not to.

## Target queries (refine together)
1. **Relational division:** for each open solicitation, find Massachusetts shops that hold *every* required, unexpired certification.
2. **Spending analysis:** DoD award dollars by manufacturing NAICS code over the last 5 years, and the share going to small businesses.
3. **Correlated subquery:** suppliers whose average award value exceeds the average for their NAICS code.
4. **Supplier concentration:** categories where the top 3 vendors capture more than 50% of spending (window functions).
5. **Capability gaps:** solicitations that no registered shop can fulfill, and which requirement blocks them most often.
6. **Match ranking:** rank shops for a given opportunity by match score.

Stored programs:
- Procedure: `match_shops(opportunity_id)` returning ranked qualified shops.
- Trigger: block inserting a bid on an ITAR-flagged opportunity from a shop without an active ITAR registration.

## Repository structure
```
/sql
  schema.sql            -- CREATE TABLE statements
  procedures.sql        -- stored procedures, functions, triggers
  queries.sql           -- the 5–6 deliverable queries with comments
/etl
  download_*.py         -- fetch raw data
  clean_*.py            -- clean and transform
  load.py               -- load into MySQL
  generate_mock.py      -- seeded mock data generator
/data
  raw/                  -- gitignored if large
  processed/
/docs
  DATA_CLEANING.md
  AI_USAGE.md
  eer_diagram.png
README.md               -- setup, run order, data sources (real vs mock)
.env.example
```

## Work plan
1. **Scope and schema:** confirm scope (region, years, NAICS codes), finalize schema, produce EER diagram.
2. **Real data ETL:** download, clean, and load USAspending, SAM.gov, and NAICS data. Document cleaning.
3. **Mock data:** generate and load realistic shops, certifications, capabilities, and bids.
4. **Queries and stored programs:** write, test, and verify each query's output makes sense.
5. **Deliverables:** export queries and output to PDF, build the 5–6 slide presentation, create the database backup and code zip.
6. **Optional:** simple UI, only after 1–5 are done.

At the start of each session, check this file and the repo state, then tell us where we are in the plan and suggest the next step.

## AI usage log (required by the course)
Maintain `docs/AI_USAGE.md`. After each significant task, add an entry with: date, task, what Claude did, what team members did or changed, and how the output was verified. This will become the "How we used AI" section of our submission, so keep it honest and specific.

## Code style
- SQL: uppercase keywords, snake_case names, one clause per line for complex queries, comment the intent of every non-trivial query.
- Python: small functions, type hints, clear names, no hardcoded credentials.
- Prefer simple, readable solutions over clever ones. We need to be able to explain every line in our presentation.
