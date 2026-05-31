# FIND (live job search)

## Step 1 — Read resume(s) AND search profile, then build the search plan

### 1A. Resume

Read every resume file in `resumes/` (skip the README and `search_profile.md`). Extract:

- **Target roles** — the user's most recent role title + adjacent titles (e.g. if they're a "Senior Data Scientist", also search "Staff Data Scientist", "Senior Product Data Scientist", "Lead Data Scientist"). Pull 3-6 titles total.
- **Top keywords / skills** — 10-15 from the skills section + repeated terms across bullets (technologies, methodologies, domain terms).
- **Default location** — the city/state from the resume's contact line.

If `resumes/` has no resume (only the README and/or `search_profile.md`), stop and tell the user to drop their resume in.

### 1B. Search profile (optional, but read every time)

Look for `resumes/search_profile.md`. If present, read the whole file as free-form prose. Apply it as overlays on the resume-inferred plan:

- **Locations:** if the search profile names locations, those replace the resume's default. (Example: profile says "Remote only" → drop the resume's city, search Remote only.)
- **Titles:** if the profile mentions specific titles or role types, narrow the title list to match. If it explicitly excludes some (e.g. "no people management"), drop manager-track titles.
- **Keywords:** add any domain interests from the profile (e.g. "climate tech", "LLM infra") to the keyword pool used for searches and scoring.
- **Salary floor:** if the profile gives a number, use it for scoring (jobs at or above floor get +1).
- **Deal-breakers:** if the profile lists exclusions (e.g. "no crypto"), drop matching jobs entirely from results, before scoring.
- **Company-size or stage preferences:** keep in mind for the score; small-company-only profile + a Fortune-500 result → drop or score low.

If the search profile contradicts the resume (e.g. resume is full of ML, profile says "I want to switch to PM"), the profile wins. The resume describes what the user *can* do; the profile describes what they *want* to do.

**No salary filter unless the search profile sets one.** Print the inferred plan as a one-line summary so the user can see it scrolling past, then proceed straight to Step 2 — no confirmation prompt.

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

Combine LinkedIn + WebSearch results into one pool.

**Rate limiting:** If LinkedIn throttles or shows a CAPTCHA mid-search, stop and tell the user. Work with results collected so far, or re-run later in smaller batches.

---

## Step 2B — Indeed (only if enabled)

Check `resumes/search_profile.md` for `Indeed: yes` (in the Platforms section) or `Discovery: enabled` (in the Indeed section). If neither is present, **skip this entire step** — Indeed is off.

### Indeed Lane A — active search

For each target role, for each location from the search plan:

```
https://www.indeed.com/jobs?q=<role>&l=<location>&fromage=7&sort=date
```

Also run each with `&l=Remote` (drop `radius`). `fromage=7` = posted in the last 7 days, `sort=date` = newest first. Paginate with `&start=10`, `&start=20` (up to 2 pages per search). Read each results page with `mcp__Claude_in_Chrome__get_page_text`. Extract Company, Role, Location, Salary (if shown), and the job URL (`/viewjob?jk=...` or `/jobs/view/...`).

**Hybrid coverage:** the local-area pass already returns hybrid roles because they are posted under the office city. Do not drop a candidate for being "Hybrid" — if the user's search profile allows Hybrid or On-site for that area, it is in-scope.

### Indeed Lane B — recommendation harvest (best-effort)

After active search, glance at logged-in Indeed for any personalized module:

- `https://www.indeed.com/`
- Any visible "Jobs for you", "Because of your profile", "Profile match", "Based on your qualifications" module.

Extract Company, Role, Location, Salary (if shown), URL, the match label, and tag source as `Indeed Recommendation`. Do not click apply. **Expect little or nothing here** — Indeed is sunsetting the desktop Recommended Jobs feature, so this lane is a bonus, not a dependency. If it is empty, move on.

### Indeed Lane C — Career Scout ingestion (mobile, user-fed)

Career Scout is Indeed's personalized recommender and it is **mobile-app only (iOS/Android)** — it cannot be driven from Chrome. Handle it as a user-in-the-loop source:

- Prompt the user near the start of `/jobs find`: "Career Scout is mobile-only. If you've run it recently, paste any roles it surfaced (links, or company + title) and I'll fold them in." If they have nothing, skip and continue (fail-soft, never block).
- When the user pastes Career Scout roles: tag each `Career Scout`. Resolve a working URL if they give only a name (via WebSearch). Run them through the same pipeline as every other source: dedup → company-site verification → score → add.
- Career Scout picks count as a personalized recommendation for scoring (+1 bonus), but still must pass the minimum score.

### Indeed pacing + fail-soft (REQUIRED)

Indeed runs Cloudflare + DataDome, stricter than LinkedIn:

- Go slow: one Indeed URL at a time, never fire all titles/pages in a burst.
- **On ANY block** (CAPTCHA / "verify you are human" / 403 / blank page): STOP the current Indeed lane, do NOT try to solve it, log `Indeed: blocked after N results`, and continue the pipeline with LinkedIn + WebSearch + any already-collected Indeed results.
- Indeed randomizes its HTML — extraction is flakier than LinkedIn. If a result's Company/Role is garbled or empty, drop it, do not guess.
- **Fallback when Indeed is blocked or thin** — backfill via WebSearch for the same titles:
  - `<title> jobs "<location>" site:indeed.com 2026`
  - `<title> <top keyword> <top keyword> jobs 2026`

Indeed is fail-soft: a blocked Indeed lane must NEVER stop LinkedIn results or the scoring/add steps from completing.

---

Combine all results (LinkedIn + WebSearch + Indeed Search + Indeed Recommendation + Career Scout) into one pool. Keep source tags through scoring and tracker notes.

---

## Step 3 — Dedup, score, rank

**Dedup:** Remove any result where Company + similar Role already exists in `job_tracker.csv`. Track skipped names to report at the end.

**Score each job (0-12) against the search plan from Step 1:**

- Title matches a target role exactly: +3
- Title is adjacent (e.g. "Staff" instead of "Senior", or "Analyst" vs "Scientist" with overlapping skills): +1.5
- Location matches a profile-allowed location OR is Remote (when profile allows Remote): +2
- Each keyword overlap (resume keywords ∪ search profile interests): +0.5 (cap at +3)
- Posted ≤7 days ago: +1
- Salary at or above the profile's stated floor (skip this rule if no floor was given): +1
- **Search profile match for company stage / size / domain** (e.g. profile says "climate tech", job is at a climate-tech company): +1
- Indeed personalized recommendation / profile-match label / Career Scout source: +1
- Matching company-site posting verified (Indeed candidate confirmed live on company careers page): +1

**Hard exclusions from the search profile** (deal-breakers like "no crypto", "no consulting"): drop the job entirely BEFORE scoring. Don't surface it in the results table.

**Company-site verification for Indeed candidates:**

For every Indeed candidate that survives the dedup:
- Search WebSearch for the same Company + Role on the company careers page.
- If a matching live company posting exists, replace the URL with the company apply URL and add `Verified company site from Indeed discovery` to Notes.
- If no direct posting is found but the Indeed posting is live and credible, keep the Indeed URL and add `Indeed-only posting; verify before apply` to Notes. Set `Apply Via=Indeed`.
- If the company site shows the role closed/missing and the Indeed post looks stale, drop it.

**Indeed recommendation rules:**
- Treat recommendations as signal, not truth. Indeed can recommend noisy, stale, sponsored, or off-target jobs.
- The +1 scoring bonus helps but never rescues a weak or off-target JD.

**Source tags to preserve in tracker Notes:**
- `Indeed Search` / `Indeed Recommendation` / `Career Scout` / `Verified company site` / `Indeed-only posting; verify before apply`

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
