# Koemi (声 × 見る)

**AI-powered customer feedback intelligence**  
*See what your customers are saying.*

## 1. The idea

Koemi is a lightweight product intelligence app for e-commerce teams. It turns customer review text and transaction data into clear, evidence-backed insights: what customers complain about, where problems are concentrated, how issues change over time, and what a product or operations team should investigate next.

The name is a coined blend of **koe** (声, voice) and **mi** (inspired by *miru*, 見る, to see). It is a brand interpretation, not a literal Japanese phrase.

**Portfolio goal:** Build and deploy a product that demonstrates data engineering, analytics, practical AI engineering, product thinking, and evaluation. This is not just a chatbot or a dashboard with LLM-generated text.

### Target users
- Product managers investigating customer pain points
- Customer experience and support leads tracking complaints
- Operations teams looking for shipping or seller problems

### Core user story
> As a product manager, I want to identify the most frequent and most consequential customer problems, inspect the evidence, and decide what to investigate first without manually reading thousands of reviews.

### A successful demo
A user opens Koemi, filters by time period or product category, sees the largest complaint themes, selects **delivery delays**, reads a few real supporting reviews, checks whether late deliveries correlate with poor ratings, and gets a short, evidence-linked summary of possible next steps.

---

## 2. Data

**Starting dataset:** Olist Brazilian E-Commerce Public Dataset  
https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

The public dataset contains approximately 100,000 orders from 2016–2018 and related order, item, product, seller, customer, and review tables. Confirm the actual downloaded files, schema, license, and row counts before writing joins or claiming metrics.

**Likely useful tables**
- `olist_order_reviews_dataset.csv` — ratings, optional review text, review dates
- `olist_orders_dataset.csv` — order dates, estimated and actual delivery dates, status
- `olist_order_items_dataset.csv` — items, sellers, prices, freight
- `olist_products_dataset.csv` — product identifiers and categories
- `olist_product_category_name_translation.csv` — product category names
- `olist_customers_dataset.csv` — location context
- `olist_sellers_dataset.csv` — seller context

**Important data realities**
- Reviews are in Portuguese. Use multilingual embeddings or documented translation, and assess performance rather than assuming English-only tools work.
- Many rows have ratings but no written review. Separate **ratings analytics** from **text analytics**, with honest denominators.
- Orders can have multiple items and occasionally multiple review records. Never join order-level review rows directly to item-level rows and then treat the result as one review per row; inspect cardinality and design aggregations deliberately.
- This is historical data, **not live monitoring**. Time trends must be labeled historical.
- A review may mention delivery or product quality, but text labels inferred by AI are not ground truth.
- Read the dataset's usage license before publishing the derived data or deploying a public demo. Attribute Olist and avoid presenting this as proprietary company data.

### Guiding analytical questions
1. What issues appear most often in written negative feedback?
2. How does average rating differ for on-time versus late deliveries?
3. Which categories or sellers appear to have disproportionate complaints (given sample sizes)?
4. Which issue themes increase across historical time periods?
5. What actions could a PM investigate based on this evidence, without implying causation?

---

## 3. MVP: build these features first

**Must-have**
1. **Overview dashboard:** total orders, reviews, written-review coverage, average rating, negative-review percentage, historical rating trend.
2. **Issues dashboard:** identify common issues from review text; show issue counts, share of *classified written reviews*, and breakdown by rating or category.
3. **Drill-down:** select an issue to view source reviews and related operational metrics (e.g., shipping delay).
4. **AI insight card:** a short grounded summary using real counts and retrieved example reviews; show evidence and avoid unsupported causal claims.
5. **Filters:** date range and product category; add seller only if joined data supports reliable aggregation.
6. **Evaluation:** labeled review sample, a simple baseline, quality metrics, and a short error analysis.

**After MVP, only if time allows**
- Ask-Koemi natural-language Q&A over precomputed metrics and retrieved evidence
- CSV upload with format validation
- Issue ranking based on transparent scoring
- User feedback on the usefulness of each insight
- Cost/latency monitoring in the UI

**Not in v1:** authentication, paid plans, multi-agent orchestration, real-time ingestion, complicated microservices, unreviewed AI-generated SQL execution, or speculative revenue-impact claims.

---

## 4. Tech stack

| Layer | Choice | Why |
|---|---|---|
| Data exploration and ETL | Python, pandas, notebooks | Quick inspection and transparent transformations |
| Storage | PostgreSQL | Real relational modeling and SQL analytics |
| Embeddings | Multilingual sentence embeddings | Portuguese-language review similarity |
| Issue detection | TF-IDF baseline; embeddings/clustering or supervised classification | Compare a simple approach with a stronger one |
| AI insight generation | An LLM API | Turn verified metrics and retrieved reviews into readable summaries |
| API | FastAPI, Pydantic | Small, typed backend |
| Frontend | React + TypeScript | A usable deployed product |
| Charts | Recharts (or equivalent) | Embedded charts in the app |
| Tests | pytest | Check ETL, APIs, and evaluation logic |
| Deployment | A simple managed frontend + backend + database setup | A live demo that a recruiter can open |

**Keep dependencies lean.** Use pgvector only if semantic retrieval in PostgreSQL is needed; a small persisted embedding index may be enough for v1. Don't add orchestration frameworks without a concrete reason.

---

## 5. Proposed system flow

```text
Olist CSVs
   |
   v
Profile + clean + validate (Python/pandas)
   |
   v
PostgreSQL (orders, reviews, items, products, derived tables)
   |                                |
   v                                v
SQL metrics                    Written-review text
   |                                |
   |                     Baseline classifier + multilingual embeddings
   |                                |
   +-----------> Issue summaries / evidence <-------+
                              |
                       Grounded LLM summary
                              |
                         FastAPI endpoints
                              |
                      React product dashboard
```

### Grounding rules
- SQL calculates totals, ratios, and trends; don't ask an LLM to invent or calculate them from memory.
- Give the LLM precomputed metrics, issue definitions, and review excerpts; require it to cite the review IDs / metric IDs provided.
- Keep generated recommendations tentative: **investigate**, **test**, or **compare** rather than claiming an intervention will work.
- Cache repeated analyses during development to control API costs.

---

## 6. Data quality and modeling

Before training models:
1. Inspect CSV schemas, nulls, duplicate IDs, date ranges, categorical values, and row counts.
2. Define each metric's unit of analysis: order, item, review, seller, or customer.
3. Build a data dictionary with join keys, one-to-many relationships, and assumptions.
4. Parse timestamps; compute delivery delay from actual minus estimated delivery date only where valid.
5. Standardize written review text while preserving the original Portuguese text.
6. Make curated SQL views / tables for safe aggregation; verify that joins do not inflate counts.
7. Document exclusions and compare row counts before/after transformations.

### Baselines before advanced AI
- Ratings analysis without NLP
- Keyword or TF-IDF + linear model for issue classification (where labeled data supports it)
- More advanced multilingual embedding-based classification or clustering

Keep the approach that works best on measured quality and complexity. Don't use XGBoost, LangChain, or agents simply to add buzzwords.

---

## 7. Evaluation plan

Create a small manually labeled set of **200–300 written reviews** sampled across ratings, time periods, and apparent topics. Decide whether a review can receive multiple issue labels (likely yes). Write clear label definitions and track uncertain cases. Avoid using test labels to tune prompts or models.

**Measure:**
- Issue detection: per-class precision/recall/F1 and macro-F1, with a baseline comparison
- Retrieval: relevance of retrieved reviews on a manually checked question set
- AI insight generation: correctness of quoted metrics, support from cited reviews, unsupported-claim rate
- Product: task success or time-to-insight in a few informal usability tests
- Engineering: API latency and estimated LLM cost per generated insight

Keep a `reports/evaluation.md` with methods, sample size, model/prompt versions, failures, and honest results. Don't claim accuracy or percentage improvements until measured.

---

## 8. Practical build sequence

### Phase 0 — Repository and scope
- [ ] Create a GitHub repo called `koemi` (or another available repo name)
- [ ] Add this project brief and a short README
- [ ] Set up Python environment, `.gitignore`, and environment-variable example
- [ ] Define 3–5 core dashboard questions and success criteria

### Phase 1 — Data foundation
- [ ] Download original Olist files and inspect them before committing to schema details
- [ ] Create an EDA notebook with 5–8 meaningful plots/tables
- [ ] Implement reproducible cleaning scripts
- [ ] Create Postgres schema and load the curated data
- [ ] Check join cardinality, missing text, date logic, and metric denominators
- [ ] Produce initial SQL metrics for the dashboard

**Deliverable:** reproducible dataset build + a concise `reports/eda.md`.

### Phase 2 — Feedback intelligence
- [ ] Define a small issue taxonomy
- [ ] Manually annotate a representative review sample
- [ ] Build and evaluate a simple baseline
- [ ] Try multilingual embeddings and compare to the baseline
- [ ] Save issue labels with confidence/uncertainty metadata
- [ ] Build evidence retrieval for a selected issue

**Deliverable:** repeatable text-analysis pipeline + evaluation report.

### Phase 3 — Backend and product
- [ ] Add FastAPI routes for overview metrics, issue trends, and review drill-down
- [ ] Implement server-side filtering
- [ ] Add AI summaries based on validated metrics and supporting review IDs
- [ ] Build a clean dashboard with a useful default view
- [ ] Add loading, empty, and error states

**Deliverable:** working end-to-end local app.

### Phase 4 — Reliability and launch
- [ ] Write tests for key SQL/ETL assumptions and API routes
- [ ] Run insight-grounding checks on a small evaluation set
- [ ] Deploy a live demo with safe environment-variable handling
- [ ] Ask 3–5 people to complete a specific PM task and record observations
- [ ] Polish README, screenshots, setup instructions, and limitations

**Deliverable:** live demo + GitHub + a one-page product case study.

---

## 9. Suggested repo structure

```text
koemi/
├── README.md
├── KOEMI_PROJECT.md
├── CLAUDE.md
├── .env.example
├── .gitignore
├── data/
│   └── README.md            # where to obtain raw data; avoid committing large files
├── notebooks/
│   └── 01_eda.ipynb
├── scripts/
│   ├── inspect_data.py
│   ├── clean_data.py
│   └── load_db.py
├── sql/
│   ├── schema.sql
│   └── metrics.sql
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── db.py
│   │   ├── routes/
│   │   └── services/
│   └── tests/
├── frontend/
│   └── ...
└── reports/
    ├── eda.md
    └── evaluation.md
```

This is a **target structure**, not a requirement to generate every file immediately. Create files as needed.

---

## 10. Claude Code instructions — coding style and workflow

The point of AI assistance is to help me build something I **actually understand and can defend**, not to manufacture an artificially sophisticated codebase. Write code at the level of a capable undergraduate data science student who knows Python, pandas, SQL, and basic software engineering. Favor good judgment over fancy abstractions.

### Rules for Claude Code

1. **Work in small increments.** Before changing code, state the next concrete objective and which files you'll edit. Implement one bounded step and show how to run/test it. Don't build the whole app in one response.
2. **Keep the code straightforward.** Prefer plain functions, familiar packages, and a simple folder layout. Avoid unnecessary helper classes, deep inheritance, dependency injection, factory patterns, and premature abstraction.
3. **Use normal naming.** Clear `snake_case` Python variables/functions; descriptive SQL aliases; natural names without trying to make every variable ultra-formal.
4. **Comment with restraint.** Explain *why* a non-obvious choice was made; don't narrate obvious syntax or leave long AI-style docstrings everywhere.
5. **No fake complexity.** Don't introduce Kubernetes, Redis, queues, agents, microservices, or elaborate design patterns unless an actual problem justifies them.
6. **Be careful with data.** Inspect real columns and sample rows before assuming a schema. Explicitly check duplicates, nulls, grain, and join inflation.
7. **Use real tests and output.** Run scripts/tests when possible. Never invent accuracy, row counts, benchmarks, metrics, or successful tool runs.
8. **Avoid silent broad rewrites.** Don't refactor unrelated modules, rename everything, or add dependencies without asking.
9. **Explain tradeoffs briefly.** Mention key alternatives, why the current approach is enough, and what would trigger a more complex solution.
10. **Make it interview-defensible.** After each feature, explain in 3–5 sentences what it does, why it's designed that way, and one limitation I should know.
11. **Prioritize readability over appearing AI-free.** Code should feel like my own because I reviewed it, understood it, and made the decisions—not because we're trying to conceal AI usage. Follow any application or class rules for disclosing AI assistance.
12. **Don't fabricate product evidence.** User feedback, business impact, metrics, and resume accomplishments must reflect actual work and measured results.

### What to avoid
- Massive scaffolding before there's working data
- 15 tiny files for one simple script
- Overly generic utilities built for imaginary future needs
- Extra logging and defensive layers in every line
- Unnecessary TypeScript generics and elaborate frontend state management
- Marketing language in technical docs
- Dashboards full of made-up KPIs

### Response format when coding
For each small task, give:
1. **What we're doing** — 1–2 sentences
2. **Implementation** — actual code/file edits
3. **How to run it** — exact command(s)
4. **How we know it works** — tests or visible output
5. **What I'd do next** — one logical next step

---

## 11. First prompt to paste into Claude Code

> I'm building **Koemi**, an AI-powered customer feedback intelligence platform. Read `KOEMI_PROJECT.md` carefully and use it as the source of truth.
>
> I want this to be a real end-to-end portfolio project for product, data, and AI engineering roles, but I also want a codebase that I understand and could reasonably have written myself as an undergraduate data science student. Keep the implementation simple, readable, and intentional. No premature architecture, giant scaffolds, needless abstractions, or fake numbers. If there's a tradeoff, explain it briefly.
>
> **Start with Phase 1 only.** First inspect the current repository and tell me what exists. Then propose the smallest first implementation step for profiling the Olist CSV data. Do not assume file schemas before inspecting them, and don't begin the React app or LLM integration yet. Make one small, testable change at a time and explain what it does.

---

## 12. Definition of done

Koemi is portfolio-ready when:
- A reviewer can open a working demo and complete the core investigation workflow.
- The README explains the problem, target user, dataset, architecture, and setup.
- The data pipeline can be rerun from source files.
- The metrics and joins are reproducible and tested.
- The AI produces traceable insights, with documented limitations and evaluation.
- The repo includes a short case study with product choices, actual results, and next steps.

### Potential resume bullets (templates — not yet earned)

- Built **Koemi**, an end-to-end customer intelligence platform integrating e-commerce transaction records and written reviews to surface recurring customer pain points.
- Developed and evaluated a multilingual NLP pipeline for issue detection, comparing a simple baseline against embedding-based methods on a manually labeled review set.
- Built PostgreSQL analytics, FastAPI services, and an interactive dashboard that connects AI-generated summaries to verified metrics and source reviews.

Replace these with actual numbers and outcomes after implementation. Avoid claiming user adoption or business impact without evidence.
