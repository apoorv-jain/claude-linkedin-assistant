# Shared context — loaded by all /jobs sub-flows

Working directory: the cloned repo root (where `job_tracker.csv` lives).
Today's date: read from system (YYYY-MM-DD).

---

## Profile

Always have `profile/profile.md` available. It contains:
- LinkedIn name (used by Step 0 verification)
- Background pitch (used by outreach DMs)
- Target roles (used by find)
- Top keywords (used by find scoring)
- Locations (used by find)
- Salary range

If `profile/profile.md` doesn't exist, stop the flow and tell the user to copy the template.

---

## When to use WebSearch

Use `WebSearch` proactively whenever Chrome returns incomplete info or you need data not on screen:
- Company headcount, funding, LinkedIn followers
- Finding a careers page or verifying a job is still open
- Finding the right LinkedIn URL for a company or person
- Any question you can answer faster with a search than by navigating Chrome

---

## Tracker columns

`Priority, Company, Role, Location, Type, Salary, Status, Applied Date, Next Action, URL, Notes, Discovered Date, Referral Needed, Referral Status, Referral Deadline, Apply Via`

**Canonical statuses:** `To Apply` → `Applied` → `Recruiter Call` → `Phone Screen` → `Onsite` → `Offer` / `Rejected` / `Withdrew`

**Priority values:** `HIGH` / `MEDIUM` / `LOW`

**Referral Status values:**
`Not Needed` / `Outreach Pending` / `Connection Pending` / `Outreach Sent` / `Got Referral` / `Declined` / `No Referral`

- `Connection Pending`: a 2nd-degree contact was identified, but the user must send the connection request manually (this repo does NOT send connection requests).
- `Outreach Sent`: a first DM was sent to a 1st-degree connection.
- `Declined`: contact replied but said they can't refer.
- `Referral Deadline`: Discovered Date + 5 days (only when Referral Needed=YES).
- `Apply Via`: `LinkedIn Easy Apply` / `Company Website` / `Blocked`.

---

## outreach/<Company>_contacts.md format

Create this file when first reaching out to anyone at a company. One file per company.

```markdown
# Outreach — <Company>

| Name | Title | LinkedIn | Stage | Last Action Date | Notes |
|---|---|---|---|---|---|
| Jane Doe | Sr Data Scientist | linkedin.com/in/... | 3. Cold DM Sent | 2026-04-22 | First DM, 1st-degree connection |
| John Smith | Recruiter | linkedin.com/in/... | 1. Connection Pending | 2026-04-22 | 2nd degree, user to send request |
```

### Conversation stages

| Stage | When |
|---|---|
| **1. Connection Pending** | 2nd-degree contact identified; **user must send the connection request** (this repo does not send them) |
| **2. Connection Accepted** | They accepted, ready for first DM |
| **3. Cold DM Sent** | First outreach message sent (only via this repo's outreach flow), awaiting reply |
| **4. Reply Received** | They replied — handled by the user, not by Claude |
| **5. Terminal** | No reply within window, user gave up, or declined (terminal) |

Stages 4 and 5 are handled by the user manually (this repo doesn't automate replies or follow-ups).

---

## Writing style — STRICT

**NEVER use em-dashes (—) in any user-facing message: LinkedIn DMs, anything sent to a contact.** Use a comma, period, parentheses, semicolon, or split into two sentences instead. This is a hard rule. Em-dashes are a tell of AI-written text.

**Fit summaries / background paragraphs are written in THIRD PERSON** as defined in `profile/profile.md`. The greeting stays first-person ("Hi <name>,"), but the background paragraph shifts to third person ("She is a senior data scientist..."). The signature is the user's first name. This applies to outreach DMs only.

This rule does NOT apply to:
- Internal documentation, tracker notes, contacts.md
- Technical strings, file paths, code

When drafting any user-facing text, scan for `—` before showing it to the user and rewrite if present.

---

## General rules

- Render all tracker data as markdown tables, never raw CSV.
- URL column mandatory for any new job row.
- Date format always YYYY-MM-DD. If creating folders: spaces → underscores, strip commas/special chars.
- **Any send / submit / click-Send in Chrome: show the user what will happen, get explicit "yes" before executing.**
- Never click the Send button on a LinkedIn DM, application form, or email without explicit user confirmation for that specific message.
- Never click the "Connect" button to send a LinkedIn connection request — this repo's outreach is first-message-only to existing 1st-degree connections.
- Use `WebSearch` any time Chrome is slow, blocked, or returns incomplete info.

---

## File uploads — not supported, do not attempt

LinkedIn DM file attach, Gmail compose, Greenhouse, Ashby, and most ATS forms block automated file upload. Computer-use can't drive the macOS file picker either. If a flow requires an attachment, **type the message and stop** — the user attaches and sends.

This repo's flows are designed to never need an upload: outreach is text-only first DMs. If the user wants to send a resume in a follow-up, they do it manually.
