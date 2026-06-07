# FIND — startup-jobs

Scrapes VC portfolio pages live, finds open roles at those companies matching your resume, and adds them to `job_tracker.csv`.

---

## Step 1 — Read resume + resolve filters

### 1A. Resume
Read every file in `resumes/` (skip `README.md` and `search_profile.md`). Extract:
- **Target roles** — most recent title + 3-5 adjacent titles.
- **Top skills / keywords** — 10-15 from the skills section + repeated terms.
- **Default locations** — every city/country mentioned in the contact line or experience section. If multiple are present (e.g. the user has worked in multiple cities), include all as candidate locations.
- **Remote preference** — if the resume mentions remote work, working from home, or distributed teams, add Remote to the effective location pool.

Never hardcode any location. Everything comes from the resume or search_profile.md.

### 1B. Search profile (optional)
If `resumes/search_profile.md` exists, apply as overlays (same rules as `jobs/find.md` Step 1B). Profile wins on conflict.

### 1C. Resolve effective filters
Priority order: explicit CLI args > search_profile.md > resume-inferred values.

| Filter | Resolution |
|---|---|
| **Locations** | `--location` arg → search_profile.md locations → locations extracted from resume in Step 1A |
| **Job types** | `--type` arg → search_profile.md type preference → all three (Remote, Hybrid, Onsite) |
| **Funds** | `--fund` arg → default scope from `_shared.md` |

No location or type is ever hardcoded. If the resume has no location and no search_profile.md exists and no `--location` arg is given, search globally (no location filter applied) and note that in the summary line.

Print: `Searching [funds] for [roles] | Locations: [x] | Types: [y]` — then proceed immediately, no confirmation.

---

## Step 2 — Scrape fund portfolio pages (Chrome + WebSearch)

For each fund in the resolved fund scope (from `_shared.md`), scrape the portfolio page to get a fresh company list. **Do not use any stored or cached list.**

### Per fund:

1. Navigate Chrome to the portfolio URL from `_shared.md`.
2. Call `mcp__Claude_in_Chrome__get_page_text` to read the page.
3. If the page is JS-heavy or returns sparse text:
   - Try `mcp__Claude_in_Chrome__scroll_mcp` to trigger lazy-loaded content.
   - If still sparse, fall back to `WebSearch`: `"<Fund name>" portfolio companies 2024 2025`
4. Extract every company entry visible: **name**, website or LinkedIn URL (if shown), sector, stage, location/country.
5. For YC: navigate `https://www.ycombinator.com/companies` with no country filter. Extract all company entries visible. Location matching happens later in Step 5 using the resume-inferred locations — do not pre-filter by country here.

Merge all companies into a single deduplicated pool (by company name).

Print: `Found <N> companies across <funds>` — then proceed.

---

## Step 3 — Sector pre-filter and prioritization

Before spending Chrome time on each company, score them for relevance to the resume:

| Signal | Points |
|---|---|
| Company sector matches resume domain (inferred from resume skills + experience, e.g. payments engineer resume + fintech company) | +2 |
| Company is remote-friendly AND effective type filter includes Remote | +1 |
| Company already has any row in `job_tracker.csv` | -2 (deprioritize; new roles may still exist) |
| Company in a deal-breaker industry from search_profile.md | Exclude entirely |

Sort descending. Take **top 40** companies. If the pool is larger, tell the user:
> "Found <N> companies in scope. Searching top 40 by sector relevance this run. Narrow with `--fund` or re-run to cover more."

---

## Step 4 — Find open roles at each company (Chrome + WebSearch in parallel)

Process companies in batches of 5.

### 4A. WebSearch (run first for all 5 in the batch simultaneously)

For each company, run queries built from the resume's target roles and keywords — not any hardcoded terms:
```
"<Company>" "<target_role_from_resume>" jobs site:linkedin.com 2025 OR 2026
```
and
```
"<Company>" "<top_skill_from_resume>" OR "<target_role_from_resume>" hiring 2025 site:linkedin.com/jobs
```

Target roles and skills come entirely from Step 1A — never from hardcoded lists.

Extract any job URLs, role titles, locations, and job types from the search results.

### 4B. Chrome (for companies where WebSearch returned at least one candidate)

Navigate to `https://www.linkedin.com/company/<slug>/jobs/` where slug is:
- Extracted from the portfolio page scrape (Step 2), OR
- Derived via WebSearch: `"<Company>" site:linkedin.com/company`

Read all listed roles with `mcp__Claude_in_Chrome__get_page_text`. If the page shows a type filter (Remote / Hybrid / On-site), apply the effective job-type filter from Step 1C before reading results.

Extract per role: **Role title, Location, Type (Remote/Hybrid/On-site), Salary (if shown), URL, Date posted**.

### 4C. Rate limiting

If Chrome shows a CAPTCHA or throttling mid-batch:
- Stop Chrome work immediately.
- Finish the remaining companies in that batch using WebSearch only.
- Report: "Chrome throttled after <N> companies. Remaining <M> companies searched via web only."

---

## Step 5 — Apply filters

From all collected roles, drop any that fail the effective filters:

- **Location filter**: role location must match one of the effective locations, OR role type is Remote (always passes if Remote is in the type filter).
- **Type filter**: role type must be in {Remote, Hybrid, Onsite} as resolved in Step 1C.
- **Deal-breakers** from search_profile.md: drop matching roles entirely before scoring.

---

## Step 6 — Dedup, score, rank

**Dedup**: Remove any result where Company + similar Role already exists in `job_tracker.csv`. Log skipped names.

**Score each role (0–10):**

| Signal | Points |
|---|---|
| Title matches a target role exactly | +3 |
| Title is adjacent (e.g. Staff vs Senior, Lead vs Principal) | +1.5 |
| Location matches effective location filter | +2 |
| Job type matches effective type filter | +2 |
| Each keyword overlap with resume skills (cap +3) | +0.5 each |
| Posted ≤14 days ago | +1 |
| Salary at or above search_profile.md floor (skip rule if no floor set) | +1 |
| Company sector matches resume domain | +0.5 |

Drop roles where total score < 4. Sort descending. Take top 20.

---

## Step 7 — Present and add

Show as a markdown table:

`# | Company | Fund | Role | Location | Type | Score | URL | Why it fits`

**Add ALL surfaced candidates to `job_tracker.csv` automatically — no per-row confirmation.** The filter + score already runs upstream.

For each row:
- Set **Priority** by score: ≥8 → HIGH, 6-7 → MEDIUM, <6 → LOW.
- Set **Status** = `To Apply`, **Discovered Date** = today.
- Set **Notes** = `Source: <Fund key>` (e.g. `Source: YC W24`, `Source: Peak XV`).
- Run referral check: navigate Chrome to the company's LinkedIn page, read follower + employee count.
  - >500 employees OR >100K followers → `Referral Needed=YES`, `Referral Status=Outreach Pending`, `Referral Deadline=today+5`
  - Otherwise → `Referral Needed=NO`, `Referral Status=Not Needed`
  - Can't determine → default NO, add `"Company size not verified"` to Notes.

After all rows are added, show:
- **Added** table (Priority, Company, Fund, Role, URL)
- **Skipped** one-liner (already in tracker)
- **Filtered out** count (failed location/type filters)

Then:
```
git commit -am "startup-jobs find: added <N> roles from <funds>"
```
