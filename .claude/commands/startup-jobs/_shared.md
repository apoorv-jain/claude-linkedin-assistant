# Shared context — loaded by all /startup-jobs sub-flows

## Fund portfolio sources

These are the live URLs Claude navigates to scrape company lists. Do NOT use stored/cached lists — always scrape fresh.

| Fund key | Portfolio URL | Notes |
|---|---|---|
| `yc` | `https://www.ycombinator.com/companies` | Supports query params: `?batch=W25` etc. No country filter by default — location matching is handled by Step 5 of `find.md` using resume data. |
| `peakxv` | `https://www.peakxv.com/portfolio/` | Peak XV (formerly Sequoia India/SEA). Scroll to load all entries. |
| `sequoia` | `https://www.sequoiacap.com/companies/` | Sequoia Capital global portfolio. |
| `a16z` | `https://a16z.com/portfolio/` | Andreessen Horowitz portfolio. |
| `accel` | `https://www.accel.com/portfolio` | Accel global portfolio. |
| `lightspeed` | `https://lsvp.com/portfolio/` | Lightspeed Venture Partners portfolio. |
| `elevation` | `https://www.elevationcapital.com/portfolio` | Elevation Capital. |
| `matrix` | `https://www.matrixpartners.in/portfolio/` | Matrix Partners. |
| `nexus` | `https://nexusvp.com/portfolio` | Nexus Venture Partners. |
| `blume` | `https://blume.vc/portfolio` | Blume Ventures. |
| `benchmark` | `https://www.benchmark.com/companies` | Benchmark Capital portfolio. |
| `greylock` | `https://greylock.com/portfolio/` | Greylock Partners portfolio. |

**Default fund scope** (when `--fund` is not specified): `yc` + `sequoia` + `a16z` + `accel`. Covers the broadest global portfolio without scraping too many pages in one run. Location and job-type filtering happens after scraping using resume data — not at the fund level.

---

## What to extract from each portfolio page

For each company entry visible on the portfolio page, extract:
- **Company name**
- **Website or LinkedIn URL** (if shown)
- **Sector / vertical** (if shown — e.g. "Fintech", "SaaS", "Healthtech")
- **Stage or batch** (if shown — e.g. "Series B", "W23")
- **Location / country** (if shown)

If a portfolio page is JavaScript-heavy and `get_page_text` returns little content, try scrolling (`mcp__Claude_in_Chrome__scroll_mcp`) or use `WebSearch` as fallback:
```
"<Fund name>" portfolio companies list 2024 2025 site:<fund-domain>
```

---

## Startup tracker — `startup_tracker.csv`

This is the dedicated tracker for `/startup-jobs`. It is separate from `job_tracker.csv`.

**Columns:**
`Priority, Company, Fund, Batch, Role, Location, Type, Salary, Status, Applied Date, Next Action, URL, Notes, Discovered Date, Referral Needed, Referral Status, Referral Deadline, Apply Via, Company Size, Funding Stage, Research Status, Platforms Researched`

**Startup-specific columns:**

| Column | Values |
|---|---|
| `Fund` | Fund key from `_shared.md` (e.g. `YC`, `Peak XV`, `a16z`) |
| `Batch` | YC batch or funding round label (e.g. `W24`, `Series B`) |
| `Company Size` | Employee count or range (e.g. `50-200`) — filled by research |
| `Funding Stage` | `Pre-seed` / `Seed` / `Series A` / `Series B` / `Series C` / `Series D+` / `IPO` / `Acquired` |
| `Research Status` | `Not Started` / `In Progress` / `Done` / `No Contact Found` |
| `Platforms Researched` | Comma-separated platforms where contacts were found (e.g. `LinkedIn, Email, Twitter`) |

**Canonical statuses:** `Exploring` → `To Apply` → `Applied` → `Recruiter Call` → `Phone Screen` → `Onsite` → `Offer` / `Rejected` / `Withdrew`

`Exploring` means the company is tracked but no open role was found. Cold outreach is still possible.

**Priority:** `HIGH` / `MEDIUM` / `LOW`

**Referral Status:** `Not Needed` / `Outreach Pending` / `Connection Pending` / `Outreach Sent` / `Got Referral` / `Declined` / `No Referral`

**Notes field convention:** always prefix with `Source: <Fund key>` (e.g. `Source: YC W24`).

---

## `startup_outreach/<Company>_contacts.md` format

One file per company. Created during `/startup-jobs research`. Updated by `/startup-jobs outreach`.

```markdown
# Startup Outreach — <Company>

| Fund | Stage | Website | LinkedIn Company Page |
|---|---|---|---|
| YC W24 | Series B | example.com | linkedin.com/company/example |

## Contacts

| Name | Title | Email | LinkedIn | Twitter/X | Research Date |
|---|---|---|---|---|---|
| Jane Smith | CEO | jane@example.com | linkedin.com/in/janesmith | @janesmith | 2026-05-15 |
| Bob Lee | VP Engineering | bob@example.com | linkedin.com/in/boblee | Not Found | 2026-05-15 |

## Outreach Log

| Name | Platform | Status | Date | Notes |
|---|---|---|---|---|
| Jane Smith | Email | Approved — Sent | 2026-05-15 | User confirmed |
| Bob Lee | LinkedIn | Draft Shown | 2026-05-15 | Awaiting approval |
```

**Contact priority order for outreach:**
1. Hiring Manager / Engineering Manager (closest to the role)
2. Recruiter / Talent Acquisition (fastest path)
3. CEO / Founder (for small companies < 50 employees only)

**Email not found:** write `Not Found` — never guess or invent an email address.

---

## Outreach platforms

| Platform | Key | How it works |
|---|---|---|
| `linkedin` | LinkedIn DM | Chrome drives LinkedIn; same flow as `jobs/outreach.md` |
| `email` | Email | Chrome opens Gmail compose (or default mail client); user clicks Send |
| `twitter` | Twitter/X DM | Chrome navigates to twitter.com/messages/compose; user clicks Send |

**Default when `--platform` not specified:** use whichever platforms have confirmed contact info for that person (email if email found, linkedin if LinkedIn URL found, etc.).

---

## Search profile — MANDATORY, read on every flow

`resumes/search_profile.md` is the source of truth for what the user wants. **Always read it at the start of every startup-jobs flow, without exception.** It overrides all resume-inferred defaults.

Apply these rules derived from the profile:

**Company type preference (in order):**
1. B2C at scale — consumer product with millions of users
2. Large established B2B — serious engineering scale (Postman, BrowserStack, Freshworks, etc.)
3. Small B2B SaaS — only if Series B+ and deep engineering problems evident

**Hard drops (drop before any scoring):**
- Consulting / IT services firms (Nagarro, Nearform, Cognizant, Infosys, Wipro, TCS, Accenture, Capgemini, HCL, etc.)
- Crypto / Web3 companies
- Pre-Series A or bootstrapped companies
- Roles with a hard 7+ year experience requirement
- Salary explicitly listed below 35 LPA (India) or $80K USD

**Scoring boosts to apply:**
- B2C at scale company → +1
- Deep engineering signal in JD (performance, architecture, platform, infrastructure) → +1
- Large established B2B → +0.5
- Salary at/above 35 LPA / $80K → +1

**Pitch sentence for all outreach** (build from resume each time, don't hardcode):
> "Tushar is a Senior Software Engineer at Procol with 4+ years in React/Next.js/TypeScript, frontend architecture, and performance optimization."
Adapt emphasis per company type: performance for B2C scale, systems/architecture for infra-adjacent, tooling for devtools.

---

## General rules (same as jobs.md)

- Render all tracker data as markdown tables, never raw CSV.
- URL column mandatory for any new job row.
- Date format always YYYY-MM-DD.
- Never use em-dashes (—) in any user-facing outgoing message.
- Use `WebSearch` any time Chrome is slow, blocked, or returns incomplete info.
- Connection requests: always "Send without a note".

---

## Outreach log — status values

Use these exact status strings in `startup_outreach/<Company>_contacts.md` Outreach Log:

| Status | Meaning |
|---|---|
| `Draft Shown` | Draft presented to user, awaiting approval |
| `Approved — Sent` | LinkedIn DM sent via Chrome |
| `Draft Loaded in Gmail` | Gmail compose opened; user clicks Send |
| `Draft Loaded` | Twitter/X compose opened; user clicks Send |
| `Connection Pending` | 2nd/3rd degree — connection request sent without note |
| `No Response` | Sent 7+ days ago, no reply seen |
| `Followed Up` | Follow-up message sent |
| `Replied` | Contact responded (positive or neutral) |
| `Declined` | Contact said no or not interested |
| `Converted` | Referral received or interview booked |

User updates `Replied`, `Declined`, `Converted` manually. Claude sets the rest automatically.

---

## Email templates

Load these templates in `outreach.md`. Fill `<placeholders>` from resume and contacts file.

**Build the pitch first** (always from `resumes/Tushar_Garg_Resume_Updated.pdf`):
- Name: Tushar
- Current role: Senior Software Engineer at Procol
- Experience: 4+ years
- Specializations: React/Next.js/TypeScript, frontend architecture, performance optimization
- Adapt emphasis per company: performance for B2C scale, architecture for platform roles, tooling for devtools

Pitch sentence shape:
> "Tushar is a Senior Software Engineer at Procol with 4+ years in React/Next.js/TypeScript, frontend architecture, and performance optimization."

Scan every draft for `—` before showing. Rewrite if found.

### Initial outreach — Recruiter

**Subject options** (show two, let user pick):
- `<Role> at <Company> — Tushar`
- `Senior Frontend Engineer interested in <Company>`

**Body:**
> Hi <First>,
>
> Tushar is a Senior Software Engineer at Procol with 4+ years in React/Next.js/TypeScript, frontend architecture, and performance optimization. I came across the <Role> opening at <Company> and it aligns closely with what I have been building.
>
> Would you be open to a quick chat or sharing more about the team?
>
> Thanks,
> Tushar
> linkedin.com/in/tushar-garg-65663b190

### Initial outreach — Engineering Manager / Hiring Manager

**Subject options:**
- `<Role> at <Company> — Tushar`
- `Senior SWE interested in <Role> at <Company>`

**Body:**
> Hi <First>,
>
> Tushar is a Senior Software Engineer at Procol with 4+ years in React/Next.js/TypeScript, frontend architecture, and performance optimization. I noticed the <Role> role at <Company> and it aligns well with what I have been building.
>
> Would you be open to a quick chat or a referral if it feels right?
>
> Thanks,
> Tushar
> linkedin.com/in/tushar-garg-65663b190

### Initial outreach — CEO (< 100 employees only)

**Subject options:**
- `<Role> at <Company> — Tushar`
- `Excited about <Company> — Tushar`

**Body:**
> Hi <First>,
>
> Tushar is a Senior Software Engineer at Procol with 4+ years in React/Next.js/TypeScript, frontend architecture, and performance optimization. I have been following <Company>'s work on <one-sentence company description from research> and I saw you're hiring for <Role>.
>
> Would love to explore if there's a fit, even briefly.
>
> Thanks,
> <user first name>
> <LinkedIn URL>

### Cold outreach — No open role (general interest)

Use when `Status=Exploring` or no role URL exists. Do NOT reference a specific opening — express general interest instead.

**Subject options:**
- `Exploring opportunities at <Company> — <user first name>`
- `<user current title> interested in <Company>`

**Body:**
> Hi <First>,
>
> <Third-person pitch>. I've been following <Company>'s work on <one-sentence description of what the company does> and would love to explore if there's a place where my background could be useful to the team.
>
> Would you be open to a brief conversation?
>
> Thanks,
> <user first name>
> <LinkedIn URL>

**LinkedIn DM — cold (no role):**
> Hi <First>, <third-person pitch>. I've been following <Company> and would love to explore if there's a fit, even if nothing is posted right now. Open to a quick chat? Thanks, <user first name>

**Twitter/X — cold (no role, max 280 chars):**
> Hi <First>, big fan of <Company>'s work on <area>. <One-sentence pitch>. Would love to connect if there's ever a fit. Thanks, <user first name>

**Custom templates:** If the user has a saved custom template in `resumes/outreach_templates.md`, use it instead. If no custom template exists and the user wants to create one, follow the template creation flow in `outreach.md` Step 2C.

---

### Follow-up — all roles (send if no reply after 7 days)

**Subject:** `Re: <original subject line>`

**Body:**
> Hi <First>,
>
> Just following up on my earlier note about the <Role> role at <Company>. Completely understand if the timing isn't right — happy to connect whenever works for you.
>
> Thanks,
> <user first name>

### LinkedIn DM follow-up (< 280 characters preferred)

> Hi <First>, just circling back on my earlier message about the <Role> role at <Company>. Still keen if there's a moment. Thanks, <user first name>

### Twitter/X follow-up (under 280 characters, required)

> Hi <First>, following up re <Company>'s <Role> opening. Happy to share more if helpful. Thanks, <user first name>

---

## Outreach timing windows

Show timing recommendations in `outreach.md` Step 5 (before presenting drafts) so the user knows when to execute sends.

| Platform | Best days | Best time (recipient TZ) | Avoid |
|---|---|---|---|
| Email | Tue, Wed, Thu | 9-11am or 4-6pm | Mon (buried), Fri, weekends |
| LinkedIn DM | Tue, Wed, Thu | 8-10am or 5-7pm | Mon morning, Fri afternoon, weekends |
| Twitter/X DM | Tue-Thu | 10am-12pm | Weekends, late evening |

**If recipient timezone is unknown:** recommend sending between 9-11am IST / 3:30-5:30am UTC / 8:30-10:30pm PT and note the ambiguity.

**If today is Friday or weekend:** add a banner to Step 5:
> "Today is <day>. Consider scheduling sends for Tuesday morning for higher open rates."

---

## Platform guardrails

Enforced by `outreach.md` before any send action.

### LinkedIn
- Do not send a DM to the same contact on the same platform within 7 days.
- If the contact is 2nd or 3rd degree, send a connection request ("Send without a note") instead of a DM. Log as `Connection Pending`.
- Do not send more than 20 DMs in a single daily run. Stop and warn if limit approached.
- InMail is out of scope — do not attempt.

### Email
- Never silently send to a `Guessed` or `Pattern-inferred` email without showing the ⚠ confidence warning and waiting for explicit user confirmation.
- If the user explicitly says "send it anyway" for a guessed email, proceed but log confidence in the Outreach Log Notes column.
- Do not send more than 10 cold emails per day.

### Twitter/X
- Hard 280-character limit — scan every draft before showing; truncate or rewrite if over.
- Do not DM a contact who has DMs restricted (Chrome will show an error). Log as `DM Restricted — skipped`.
- Do not contact the same Twitter handle within 14 days.

### General
- Never attempt file uploads or attachments through Chrome automation.
- Stop immediately if Chrome shows a CAPTCHA or rate-limit notice. Report what completed and what is still pending.
- All draft content is shown to the user before any Chrome send action. No exceptions.
