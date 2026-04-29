# FIND (live job search)

## Step 1 — Read the user's resume(s) and infer search profile

Read every resume file in `resumes/` (skip the README). Extract:

- **Target roles** — the user's most recent role title + adjacent titles (e.g. if they're a "Senior Data Scientist", also search "Staff Data Scientist", "Senior Product Data Scientist", "Lead Data Scientist"). Pull 3-6 titles total.
- **Top keywords / skills** — 10-15 from the skills section + repeated terms across bullets (technologies, methodologies, domain terms).
- **Default location** — the city/state from the resume's contact line.

If `resumes/` has nothing useful (only the README), stop and tell the user to drop their resume in.

Ask the user once at the start of the run (single line):

> "Search profile inferred from resume: titles=<list>, keywords=<list>, location=<city/state>+Remote. Override anything? [Enter to accept]"

Common overrides: switch the city, restrict to Remote only, narrow titles. Accept whatever they say verbatim, then proceed.

**No salary filter by default** — salary expectations aren't on most resumes. If the user wants a salary floor in scoring, they'll mention it at the override prompt.

---

## Step 2 — Search (Chrome + WebSearch in parallel)

**LinkedIn via Chrome — for each target role, for each location:**

```
https://www.linkedin.com/jobs/search/?keywords=<role>&location=<location>&f_TPR=r604800&sortBy=DD
```

(`f_TPR=r604800` = last 7 days · `sortBy=DD` = date posted descending)

Read results. Extract per job: Company, Role, Location, Type (Remote/Hybrid/On-site), Salary (if shown), URL. Paginate up to 2 pages per search.

**WebSearch sweep (run in parallel with the Chrome searches):**

Build 3-5 queries combining target titles, top keywords, and locations inferred from the resume. Examples:
- `"<target title>" jobs "<location>" site:linkedin.com OR site:indeed.com 2026`
- `"<target title>" "<keyword>" "<keyword>" Remote jobs 2026`
- `"<adjacent title>" "<keyword>" "<location>" 2026`

Combine all results into one pool.

**Rate limiting:** If LinkedIn throttles or shows a CAPTCHA mid-search, stop and tell the user. Work with results collected so far, or re-run later in smaller batches.

---

## Step 3 — Dedup, score, rank

**Dedup:** Remove any result where Company + similar Role already exists in `job_tracker.csv`. Track skipped names to report at the end.

**Score each job (0-10) against the inferred search profile:**

- Title matches a target role exactly: +3
- Title is adjacent (e.g. "Staff" instead of "Senior", or "Analyst" vs "Scientist" with overlapping skills): +1.5
- Location matches the user's location OR is Remote: +2
- Each keyword overlap with the resume's top keywords: +0.5 (cap at +3)
- Posted ≤7 days ago: +1
- Salary in user's stated range (only if the user provided one at the override prompt): +1

Drop jobs where total score < 4. Sort descending. Take top 20.

---

## Step 4 — Present and add (automatic)

Show as a markdown table for visibility:
`# | Company | Role | Location | Salary | Score | URL | Why it fits`

**Then add ALL surfaced candidates to the tracker automatically — no per-candidate confirmation.** The dedup + score filter already runs upstream, so anything that reaches this step should be added. The user can prune later.

For each candidate → run the ADD flow automatically:
- Pre-fill Company, Role, Location, URL from the result
- Set Priority by score: score ≥8 → HIGH, 6-7 → MEDIUM, <6 → LOW
- Run referral check (size threshold from `add.md`)

After adding, show:
- A summary table of the rows added (Priority, Company, Role, URL)
- A one-line **Skipped** list of companies already in the tracker that were deduped
