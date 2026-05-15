# CHECK — startup-jobs dashboard

Reads `startup_tracker.csv` and `startup_outreach/` and prints a structured daily snapshot. No Chrome needed; no writes to any file.

---

## Step 1 — Load tracker

Read `startup_tracker.csv`. If empty (header only), tell the user:
> "No startup jobs tracked yet. Run `/startup-jobs find` to discover roles."
Then stop.

---

## Step 2 — Pipeline overview

Print a summary table grouped by Status:

| Status | Count |
|---|---|
| To Apply | N |
| Applied | N |
| Recruiter Call | N |
| Phone Screen | N |
| Onsite | N |
| Offer | N |
| Rejected / Withdrew | N |

Then a breakdown by Fund:

| Fund | Total | To Apply | Applied | Active |
|---|---|---|---|---|

---

## Step 3 — Action items (ordered by urgency)

### 3A. Overdue referral deadlines
Rows where `Referral Needed=YES` AND `Referral Deadline < today`. Show in red (bold):
> **OVERDUE** — <Company> (<Role>) referral deadline was <date>

### 3B. Referral deadlines in the next 5 days
Rows where `Referral Needed=YES` AND `Referral Deadline` is within 5 days. Show with days remaining.

### 3C. Research not started
Rows where `Research Status=Not Started` AND `Priority=HIGH`. These need the research flow run first before outreach is possible.

### 3D. Research done, outreach not sent
Rows where `Research Status=Done` AND `Referral Status=Outreach Pending`. Ready to run `/startup-jobs outreach`.

### 3E. Stale "To Apply" entries
Rows where `Status=To Apply` AND `Discovered Date` is more than 14 days ago. Flag as possibly expired.

### 3F. Applied — no activity
Rows where `Status=Applied` AND `Applied Date` is more than 10 days ago with no update. Suggest a follow-up check.

### 3G. Unconfirmed drafts — mark as sent or skipped
Read all `startup_outreach/<Company>_contacts.md` files. Find Outreach Log entries where Status is `Draft Loaded` or `Draft Loaded in Gmail`.

These were loaded into compose windows but the user hasn't confirmed whether they were sent. For each:
> UNCONFIRMED — <Company> · <Name> (<Title>) via <Platform> · loaded <date>

Then prompt:
> "Did you send these? Reply with `all sent`, `none`, or `<Name>: sent/skipped` to update the log. Follow-up dates are set from the confirmation date."

Process the user's response and update statuses before moving on. (`Approved — Sent` if sent, `User Skipped — not sent` if not.)

### 3H. Follow-ups due
Read all `startup_outreach/<Company>_contacts.md` files. Using today's date from the `currentDate` context variable, find Outreach Log entries where:
- Status is `Approved — Sent` AND
- `days_since = today - log_date >= 7` (YYYY-MM-DD arithmetic) AND
- No later log row exists for that contact on that platform (no `Followed Up`, `Replied`, `Declined`, `Converted`)

For each, print the exact days elapsed:
> FOLLOW-UP DUE — <Company> · <Name> (<Title>) via <Platform> · sent <log_date> (<days_since> days ago)

If none: print "No follow-ups due today."

### 3I. Already-outreached dedup
Read all outreach log files. List any contact where `Referral Status=Outreach Sent` in the tracker but no `Replied` or `Converted` entry in the log. These are in-flight — the user should update the log if they received a response.

---

## Step 4 — Outreach log summary

For each company that has a `startup_outreach/<Company>_contacts.md` file, read it and summarize:

| Company | Contacts Found | LinkedIn | Email | Twitter | Last Action |
|---|---|---|---|---|---|

Highlight any contacts where a draft was shown but not yet confirmed (Status = `Draft Shown`).

---

## Step 5 — Suggested next actions

Based on the above, print a numbered list of the top 3 things to do right now, in order of urgency:

1. Run `/startup-jobs outreach <Company>` for any overdue referral companies.
2. Run `/startup-jobs research <Company>` for HIGH priority companies with `Research Status=Not Started`.
3. Run `/startup-jobs find` if no new roles were added in the last 7 days.

---

## Step 6 — Daily checklist

Print a quick-scan checklist the user can work through each day. Pull counts from the analysis above.

```
DAILY CHECKLIST — <today YYYY-MM-DD>

[ ] Mark sent/skipped: confirm any unconfirmed draft windows from last session  →  <N unconfirmed, or "none">
[ ] Check LinkedIn notifications for reply DMs
[ ] Check Gmail for replies to startup outreach emails
[ ] Check Twitter/X DMs for replies
[ ] Send follow-ups on messages sent 7+ days ago with no reply               →  <N due, or "none">
[ ] Run research for HIGH priority companies                                  →  <N not started, or "none">
[ ] Load outreach drafts for research-complete companies (you click Send)     →  <N ready, or "none">
[ ] Apply to top "To Apply" roles before deadline                            →  <N overdue, or "none">
[ ] Update log for any recent replies or interviews (Replied / Converted)     →  <N in-flight, or "none">
```

Items with `→ none` can be skipped today. Items with a count are actionable — run the matching `/startup-jobs` sub-command to address them.
