# ADD (new job)

Collect — ask for anything missing:

1. Company + Role
2. Job URL (**mandatory** — search if not provided: `WebSearch` "<Company> <Role> job opening 2026")
3. Location + Type (Remote / Hybrid / On-site)
4. Salary (optional)
5. Priority (HIGH / MEDIUM / LOW) — ask if unclear
6. Notes (optional)

---

## Auto-determine Referral Needed

Navigate Chrome to the company's LinkedIn page. Read follower count + employee count.

- If Chrome fails → `WebSearch` "<Company> LinkedIn followers employees 2026".
- **>500 employees OR >100K followers** → `Referral Needed=YES`, `Referral Status=Outreach Pending`, `Referral Deadline=today+5`
- Below both thresholds → `Referral Needed=NO`, `Referral Status=Not Needed`
- Still unclear → ask the user.

---

Set `Discovered Date=today`, `Status=To Apply`. Append the row to `job_tracker.csv`. Show the new row as a markdown table.

**Handoff:** If `Referral Needed=YES` → ask: "Want to start outreach now? (runs `/jobs outreach <Company>`)"
If no → leave as `Outreach Pending` and done.
