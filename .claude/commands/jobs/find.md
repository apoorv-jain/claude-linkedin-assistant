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

Run ALL sources below in parallel. Combine results into one pool before scoring.

---

### 2A — LinkedIn (Chrome)

For each target role + location:
```
https://www.linkedin.com/jobs/search/?keywords=<role>&location=<location>&f_TPR=r604800&sortBy=DD
```
(`f_TPR=r604800` = last 7 days · `sortBy=DD` = date posted descending)

Paginate up to 2 pages per search. Extract per job: Company, Role, Location, Type, Salary (if shown), URL.

---

### 2B — Instahyre (Chrome)

```
https://www.instahyre.com/search-jobs/?designation=<role>&location=<location>
```
Instahyre focuses on product companies and funded startups in India. Good signal-to-noise for Tushar's target. Extract same fields.

---

### 2C — Naukri (WebSearch — do not navigate directly, it blocks automation)

Use WebSearch:
```
site:naukri.com "senior software engineer" OR "senior frontend engineer" React TypeScript <location> 2026
```
Extract job titles, companies, and URLs from search results.

---

### 2D — Work at a Startup / YC jobs (Chrome)

```
https://www.workatastartup.com/companies?demographic=any&hasEquity=any&hasSalary=any&industry=any&interviewProcess=any&jobType=fulltime&layout=list-compact&role=eng&sortBy=created_desc&tab=any&usVisaNotRequired=any
```
YC-backed startups only. Scroll to load results. Extract company name, role, location, URL.

---

### 2E — Wellfound (Chrome)

```
https://wellfound.com/jobs
```
Filter by role: "Frontend Engineer" or "Software Engineer". Extract company, role, location, funding stage, URL.

---

### 2F — OnlyFrontendJobs (Chrome)

```
https://www.onlyfrontendjobs.com/jobs?tech_stack=React%2CNext.js%2CTypeScript%2CJavaScript%2CHTML%2CCSS%2CTailwind+CSS%2CSCSS%2FSass%2CREST+APIs%2CRedux%2CReact+Query%2CWebpack%2CVite%2CPlaywright%2CAWS%2CCI%2FCD%2CGit%2CGraphQL%2CNode.js%2CExpress%2CTesting+Library%2CVercel%2CFirebase%2CSupabase%2CStorybook&job_type=Remote&experience_level=Mid
```
Pre-filtered for Tushar's exact stack (React, Next.js, TypeScript, Redux, React Query, Vite, AWS, CI/CD, Node.js). Remote + Mid-level filter already applied. High precision — prioritise these results.

---

### 2G — Hacker News "Who is Hiring?" (WebSearch)

Direct from founders, no recruiters, high signal. Use WebSearch:
```
site:hnhiring.com React TypeScript "senior" frontend 2026
```
Also check: `https://hnhiring.com/technologies/react` — pre-filtered by React.
Extract company, role, location, URL. Prefer posts from current month.

---

### 2H — levels.fyi/jobs (Chrome)

Salary-transparent listings from well-funded companies. Navigate to:
```
https://www.levels.fyi/jobs?jobType=fulltime&roles=Frontend+Engineer&roles=Software+Engineer&locations=India&locations=Remote
```
Strong signal for compensation benchmarking. Extract company, role, salary, location, URL.

---

### 2I — Cutshort (Chrome)

Strong for India product companies and funded startups. Navigate to:
```
https://cutshort.io/jobs/frontend-developer-jobs
```
Filter by React / TypeScript. Extract company, role, location, funding stage, URL.

---

### 2J — arc.dev (WebSearch)

US-paying remote roles for engineers in India. Use WebSearch:
```
site:arc.dev "senior frontend engineer" OR "senior software engineer" React TypeScript remote 2026
```
Good for roles that pay US/Europe salaries to remote India engineers. Extract company, role, salary, URL.

---

### 2K — RemoteOK (WebSearch)

Async remote startups, often US/Europe-based. Use WebSearch:
```
site:remoteok.com react typescript senior frontend engineer 2026
```
Extract company, role, salary (often listed), URL.

---

### 2L — Peerlist Jobs (Chrome)

Good for Indian tech product companies and funded startups. Navigate to:
```
https://peerlist.io/jobs?role=Frontend+Engineer&stack=React
```
Extract company, role, location, URL.

---

### 2M — WebSearch sweep

Build 3-5 queries combining target titles, top keywords, and locations:
- `"senior software engineer" React TypeScript "<location>" site:linkedin.com OR site:wellfound.com 2026`
- `"senior frontend engineer" "Next.js" "TypeScript" India Remote jobs 2026`
- `"senior software engineer" React B2C funded startup India 2026`
- `"senior frontend engineer" React TypeScript remote "India" site:greenhouse.io OR site:lever.co 2026`

---

**Rate limiting:** If any site throttles or shows a CAPTCHA, skip it and continue with others. Report what was skipped at the end.

---

## Step 3 — Dedup, score, rank

**Dedup:** Remove any result where Company + similar Role already exists in `job_tracker.csv`. Track skipped names to report at the end.

### Hard drops (apply BEFORE scoring — do not surface these at all)

- Consulting firms or agencies: Nagarro, Nearform, Cognizant, Infosys, Wipro, TCS, Accenture, Capgemini, HCL, Mphasis, LTIMindtree, and similar body-shop / IT services firms
- Crypto / Web3 companies
- Small B2B SaaS: pre-Series B, no consumer surface, limited engineering scale
- Bootstrapped or pre-Series A companies
- Roles with a hard 7+ year experience requirement explicitly stated
- Salary explicitly listed below 35 LPA (India) or below $80K USD (US/Europe)

If uncertain whether a company is a deal-breaker, use WebSearch to check funding stage and business model before deciding.

### Scoring (0-10)

| Signal | Points |
|---|---|
| Title matches target role exactly (Senior Software Engineer, Senior Frontend Engineer, SDE II/III) | +3 |
| Title is adjacent (Lead, Staff, or slightly different phrasing with same seniority) | +1.5 |
| Location matches India / Remote / US-Europe remote open to India | +2 |
| Each keyword overlap from resume (React, Next.js, TypeScript, Node.js, Performance, Design System, AWS, CI/CD, Redux, React Query, GenAI, Vite) | +0.5 each, cap +3 |
| Posted ≤7 days ago | +1 |
| Salary at or above 35 LPA / $80K USD | +1 |
| B2C company at scale (Swiggy, CRED, Zepto, Razorpay, Meesho, PhonePe, Groww, Zomato, Flipkart, Airbnb, Uber, etc.) | +1 |
| Deep engineering signal in JD (performance, architecture, platform, infrastructure, scale mentioned) | +1 |
| Large established B2B with strong eng culture (Postman, BrowserStack, Freshworks, Chargebee, Atlassian, Notion, Figma, Linear, etc.) | +0.5 |

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
