# RESEARCH — deep contact discovery

Finds decision-makers at a target company (CEO, Hiring Manager, Engineering Manager, Recruiter) across LinkedIn, email, and Twitter/X. Writes results to `startup_outreach/<Company>_contacts.md` and updates `startup_tracker.csv`. Does NOT send any messages.

---

## Step 1 — Pick a company

**If called with an argument** (`/startup-jobs research Stripe`): use that company. Look it up in `startup_tracker.csv` for context (Fund, Role, Funding Stage). If not in the tracker, proceed with just the company name.

**If called with no argument**: read `startup_tracker.csv`. Pick the highest-priority company where `Research Status=Not Started` and `Referral Needed=YES`. Tiebreaker: soonest Referral Deadline. Print:
> `Researching: <Company> — <Role> (deadline: <date>)`

If no candidates exist, tell the user and stop.

---

## Step 2 — Discover company basics

Use WebSearch and Chrome to fill in any gaps:

- **Website**: `"<Company>" official site`
- **LinkedIn company page**: `"<Company>" site:linkedin.com/company`
- **Email domain**: infer from the website (e.g. `stripe.com` → emails are likely `@stripe.com`)
- **Company size**: check LinkedIn company page (employee count) or Crunchbase
- **Funding stage**: check Crunchbase or TechCrunch: `"<Company>" funding 2024 2025`

Update `startup_tracker.csv`: fill in `Company Size` and `Funding Stage` if they were blank.

---

## Step 3 — Find target contacts

Search for contacts in this priority order. Stop adding once you have at least 3 confirmed contacts with at least one reachable channel each.

### Priority 1 — Hiring Manager / Engineering Manager

Search LinkedIn via Chrome:
```
https://www.linkedin.com/search/results/people/?currentCompany=["<company_id>"]&keywords=engineering+manager+OR+hiring+manager+OR+engineering+lead+OR+VP+engineering
```

Also WebSearch:
```
"<Company>" "engineering manager" OR "VP Engineering" OR "Head of Engineering" site:linkedin.com/in
```

### Priority 2 — Recruiter / Talent Acquisition

```
https://www.linkedin.com/search/results/people/?currentCompany=["<company_id>"]&keywords=recruiter+OR+talent+acquisition+OR+technical+recruiter
```

### Priority 3 — CEO / Founder (only for companies with <100 employees)

```
https://www.linkedin.com/search/results/people/?currentCompany=["<company_id>"]&keywords=CEO+OR+founder+OR+co-founder
```

For each contact found, extract: **Name, Title, LinkedIn URL**.

---

## Step 4 — Find email addresses

For each contact discovered in Step 3, try each source in order. Stop as soon as you find a confirmed or high-confidence email.

### Source A — Company website team/about page

Navigate Chrome to `<company-website>/team` or `<company-website>/about`. Read names and any email addresses shown.

### Source B — Hunter.io lookup

Navigate Chrome to:
```
https://hunter.io/search/<company-domain>
```
Read any email addresses and patterns shown (e.g. `{first}@company.com`). If a pattern is shown, apply it to construct the contact's email — mark it as `Pattern-inferred`.

### Source C — WebSearch

```
"<Full Name>" "<Company>" email
"<Full Name>" "<Company>" contact
"<Full Name>" site:linkedin.com email
```

### Source D — GitHub

```
"<Full Name>" "<Company>" site:github.com
```
Navigate to their GitHub profile if found. Check the profile bio and public email field.

### Source E — Email pattern guess (last resort)

If Sources A-D failed but you know the company's email domain and have seen examples of other employees' email patterns:
- Apply the inferred pattern (e.g. `firstname.lastname@company.com`).
- Mark the email as `Guessed (pattern: <pattern>)` — never present a guessed email as confirmed.

**If no email found after all sources:** write `Not Found`. Never invent an email.

---

## Step 5 — Find Twitter/X handles

For each contact, run:
```
"<Full Name>" "<Company>" site:twitter.com OR site:x.com
```
or
```
"<Full Name>" Twitter OR X "@"
```

Navigate Chrome to the Twitter/X profile if found. Verify the bio mentions the company before saving the handle.

If not found: write `Not Found`.

---

## Step 6 — Write contact file

Create or update `startup_outreach/<Company>_contacts.md` using the format from `_shared.md`.

For each contact, record:
- Name, Title
- Email (with confidence label: `Confirmed` / `Pattern-inferred` / `Guessed (pattern: ...)` / `Not Found`)
- LinkedIn URL
- Twitter/X handle
- Research Date = today

If the file already exists, read it first and merge — do not duplicate existing contacts. Update stale entries if new info was found.

---

## Step 7 — Update tracker

In `startup_tracker.csv`, for the researched company:
- Set `Research Status=Done` (or `No Contact Found` if Step 3 found nobody)
- Set `Platforms Researched` to the comma-separated list of platforms where at least one contact was found (e.g. `LinkedIn, Email`)

---

## Step 8 — Summary

Print:

```
Research complete — <Company>

Contacts found:
  1. Jane Smith (CEO) — Email: confirmed · LinkedIn: yes · Twitter: yes
  2. Bob Lee (VP Engineering) — Email: pattern-inferred · LinkedIn: yes · Twitter: not found
  3. Alice Chen (Recruiter) — Email: not found · LinkedIn: yes · Twitter: not found

Run `/startup-jobs outreach <Company>` to draft and review messages.
```

**If no open role was found** (Status=Exploring in tracker, or no job URL was set): add this line to the summary:

> "No open role found at <Company>. Run `/startup-jobs outreach <Company>` anyway to send a cold general-interest message — I'll use cold templates and can help you write a custom one if you'd like."

Do NOT draft or send any messages here. Research only.
