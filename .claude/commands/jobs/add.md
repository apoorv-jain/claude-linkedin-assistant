# ADD (new job)

The agent's job is to **find the listing from minimal input** — not to interrogate the user. Default behavior: take whatever the user gave you (could be just a URL, just "Company + Role", or just "Company"), and fill in the rest yourself via Chrome + WebSearch.

## Step 1 — Resolve to a job URL

| What the user gave | What the agent does |
|---|---|
| Just a URL | Navigate to it. Scrape Company, Role, Location, Type, Salary from the posting. |
| Company + Role | Run `WebSearch` for `"<Company>" "<Role>" jobs site:linkedin.com OR site:greenhouse.io OR site:ashbyhq.com 2026`. Pick the most recent matching listing. Open it in Chrome and scrape the same fields. If multiple plausible matches (>1 active listing for that title at that company), pick the most recently posted; print a one-line note "Multiple matches, picked most recent: <URL>". |
| Just Company | Run `WebSearch` for `"<Company>" careers OR jobs 2026` to get the careers page. Navigate Chrome there. List the open roles, ask the user "Which role?" with a numbered list. Then continue as in row 2. |

If the search returns nothing usable for the Company+Role combo (no live listing, only old/closed posts), tell the user clearly: "Couldn't find a live listing for `<Role>` at `<Company>`. Paste the job URL and I'll add it." Wait for the URL, then continue.

## Step 1B — Deal-breaker check (run before adding)

Before filling any row, verify the company is not a deal-breaker. Use WebSearch if needed to check funding stage and business model.

**Hard stops — warn the user and do NOT add to tracker:**
- Consulting / IT services firm (Nagarro, Cognizant, Infosys, Wipro, TCS, Accenture, Capgemini, HCL, etc.)
- Crypto / Web3 company
- Pre-Series A or bootstrapped startup
- Role explicitly requires 7+ years of experience

If it's a deal-breaker, tell the user:
> "Skipping `<Company>` — it matches a deal-breaker from your search profile (`<reason>`). If you want to add it anyway, tell me to override."

Wait for explicit override before proceeding.

**Yellow flags — add but note in Notes column:**
- Small B2B SaaS (Series A, limited scale, no consumer surface) → Notes: `"Small B2B — verify depth of eng problems before applying"`
- Salary explicitly below 35 LPA → Notes: `"Salary below 35 LPA floor"`
- Role requires 6-7 years (borderline) → Notes: `"Experience req borderline — worth applying"`

## Step 2 — Auto-fill the row

From the resolved listing, extract and fill:

- **Company, Role, URL, Location, Type, Salary** — straight from the page.
- **Priority** — default to `MEDIUM`. Bump to `HIGH` if: title is a strong match for Senior SWE/Frontend Engineer, company is B2C at scale or large established B2B, or salary is at/above 35 LPA floor. Drop to `LOW` if the role is borderline-relevant or company is small B2B.
- **Notes** — leave blank unless flagged by Step 1B or user provided something.

Do NOT ask the user for any of these. If a field can't be scraped (e.g. salary not shown), leave it blank.

---

## Auto-determine Referral Needed

Navigate Chrome to the company's LinkedIn page. Read follower count + employee count.

- If Chrome fails → `WebSearch` "<Company> LinkedIn followers employees 2026".
- **>500 employees OR >100K followers** → `Referral Needed=YES`, `Referral Status=Outreach Pending`, `Referral Deadline=today+5`
- Below both thresholds → `Referral Needed=NO`, `Referral Status=Not Needed`
- **Still unclear (Chrome AND WebSearch couldn't give a clear signal)** → default to `Referral Needed=NO`, `Referral Status=Not Needed`. Do NOT ask the user. Add a note in the row's `Notes` column: `"Company size not verified, defaulted to NO referral"` so the user can flip it later if they realize otherwise.

---

Set `Discovered Date=today`, `Status=To Apply`. Append the row to `job_tracker.csv`. Show the new row as a markdown table.

**Handoff (auto, no ask):** If `Referral Needed=YES` → immediately run `/jobs outreach <Company>` (read `outreach.md` and execute Step 1 onward with the new company as the argument). No confirmation prompt. The user already triggered `/jobs add`, so kicking off outreach for big companies is the expected next step.

If `Referral Needed=NO` → done. The row sits in the tracker for the user to apply to manually.
