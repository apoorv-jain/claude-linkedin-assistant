# FIND (live job search)

## Step 1 — Read the user's profile

Read `profile/profile.md` and extract:
- Target roles (job titles to search for)
- Top keywords / skills
- Locations
- Salary range

If the profile is missing or sparse, stop and ask the user to fill it in first.

---

## Step 2 — Search (Chrome + WebSearch in parallel)

**LinkedIn via Chrome — for each target role, for each location:**

```
https://www.linkedin.com/jobs/search/?keywords=<role>&location=<location>&f_TPR=r604800&sortBy=DD
```

(`f_TPR=r604800` = last 7 days · `sortBy=DD` = date posted descending)

Read results. Extract per job: Company, Role, Location, Type (Remote/Hybrid/On-site), Salary (if shown), URL. Paginate up to 2 pages per search.

**WebSearch sweep (run in parallel with the Chrome searches):**

Build 3-5 queries combining target titles, top keywords, and locations from the profile. Examples:
- `"<target title>" jobs "<location>" site:linkedin.com OR site:indeed.com 2026`
- `"<target title>" "<keyword>" "<keyword>" Remote jobs 2026`
- `"<adjacent title>" "<keyword>" "<location>" 2026`

Combine all results into one pool.

**Rate limiting:** If LinkedIn throttles or shows a CAPTCHA mid-search, stop and tell the user. Work with results collected so far, or re-run later in smaller batches.

---

## Step 3 — Dedup, score, rank

**Dedup:** Remove any result where Company + similar Role already exists in `job_tracker.csv`. Track skipped names to report at the end.

**Score each job (0-10) against the user's profile:**

- Title matches a target role exactly: +3
- Title is adjacent (e.g. "Staff" instead of "Senior", or "Analyst" vs "Scientist" with overlapping skills): +1.5
- Location matches one of the user's preferred locations OR is Remote: +2
- Each keyword overlap with profile's top keywords: +0.5 (cap at +3)
- Salary in user's range: +1
- Posted ≤7 days ago: +1

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
