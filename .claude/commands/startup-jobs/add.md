# ADD — manually add a startup to the tracker

Adds a startup by company name or job URL, runs the referral check, deduplicates against `startup_tracker.csv`, writes the row, and offers to kick off research or outreach.

---

## Step 1 — Resolve the company

**If called with a URL** (`/startup-jobs add https://...`):
Navigate Chrome to the URL. Extract:
- Company name
- Role title
- Location
- Job type (Remote / Hybrid / Onsite)
- Apply via (LinkedIn / Greenhouse / Lever / Company Site)

**If called with a company name** (`/startup-jobs add Stripe`):
Ask for the role URL. If the user doesn't have one:
1. Run WebSearch:
   ```
   "<Company>" software engineer jobs site:linkedin.com OR site:greenhouse.io OR site:lever.co
   ```
2. Also check the company's careers page via Chrome (navigate to `<company-website>/careers` or `<company-website>/jobs`).
3. If a role is found: show it and confirm before using.
4. If no role is found anywhere: tell the user:
   > "No open roles found at <Company> on LinkedIn, Greenhouse, Lever, or their careers page. Adding as 'Exploring' — you can still research contacts and send a cold message."
   Set `Status=Exploring`, `Role=blank`, `URL=blank`. Skip Step 2's Role and URL fields.

**If called with no argument**: ask:
> Company name or job URL?

---

## Step 2 — Fill in missing details

For any field not resolved from the URL, prompt the user once:

> **Adding: <Company>**
>
> Fill in what I couldn't find (leave blank to skip):
> - Role: <resolved or blank>
> - Location: <resolved or blank>
> - Type: Remote / Hybrid / Onsite
> - Fund (which VC backs them, if known): ??
> - Priority: HIGH / MEDIUM / LOW
> - Salary range (if listed): ??

Accept partial answers. Leave unknown fields blank — do not substitute "??" into the CSV.

---

## Step 2B — Deal-breaker check (run before writing anything)

Before filling in any row, verify the company is not a deal-breaker. Use WebSearch to check funding stage and business model if not obvious.

**Hard stops — warn the user, do NOT add to tracker:**
- Consulting / IT services firm (Nagarro, Cognizant, Infosys, Wipro, TCS, Accenture, Capgemini, HCL, Nearform, etc.)
- Crypto / Web3 company
- Pre-Series A or bootstrapped startup
- Role explicitly requires 7+ years of experience

If triggered, tell the user:
> "Skipping `<Company>` — matches a deal-breaker from your search profile (`<reason>`). Tell me to override if you still want to add it."

Wait for explicit override before proceeding.

**Yellow flags — add but note in Notes column:**
- Small B2B SaaS (Series A, no consumer surface) → Notes: `"Small B2B — verify eng depth before applying"`
- Salary listed below 35 LPA → Notes: `"Below 35 LPA salary floor"`
- Role requires 6-7 years → Notes: `"Experience req borderline"`

## Step 3 — Dedup check

Scan `startup_tracker.csv` for an existing row where Company AND Role both match (case-insensitive, partial match OK).

- **Match found**: tell the user:
  > "This role is already tracked (Status: <status>). Update it, or add as a separate entry?"
  - `update` → modify the existing row in place; ask which fields to change.
  - `add` → proceed and write a new row.

- **No match**: proceed.

---

## Step 4 — Referral check

Same logic as `CLAUDE.md`:
- Company > 500 employees OR > 100K LinkedIn followers → `Referral Needed=YES`
- `Referral Deadline = today + 5 days`
- Otherwise → `Referral Needed=NO`

Print:
> "Referral recommended — deadline <date>." (if YES)

---

## Step 5 — Write to startup_tracker.csv

Append a new row:

| Column | Value |
|---|---|
| Priority | as resolved |
| Company | company name |
| Fund | as resolved (blank if unknown) |
| Batch | blank |
| Role | role title |
| Location | as resolved |
| Type | Remote / Hybrid / Onsite |
| Salary | as resolved (blank if unknown) |
| Status | `To Apply` (role found) or `Exploring` (no role found) |
| Applied Date | blank |
| Next Action | `Research contacts` |
| URL | job URL (blank if no role found) |
| Notes | `Added manually` |
| Discovered Date | today (YYYY-MM-DD) |
| Referral Needed | YES / NO |
| Referral Status | `Outreach Pending` if YES, else `Not Needed` |
| Referral Deadline | computed date (blank if NO) |
| Apply Via | inferred from URL |
| Company Size | blank |
| Funding Stage | blank |
| Research Status | `Not Started` |
| Platforms Researched | blank |

Print the new row as a markdown table for confirmation.

---

## Step 6 — Offer next steps

> Added <Company>. What next?
>
>   `r` — run research now (find CEO / HM / EM contact info)
>   `o` — go to outreach (only if contacts file already exists)
>   `d` — done, I'll handle it manually
>
> Or press Enter to exit.

- `r` → read and execute `research.md` for this company.
- `o` → read and execute `outreach.md` for this company.
- `d` / Enter → stop.
