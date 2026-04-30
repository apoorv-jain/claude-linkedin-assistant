# UPDATE (status change)

## Empty-tracker guard

Read `job_tracker.csv`. If it has 0 data rows (just the header), STOP: "Your tracker is empty. Run `/jobs find` or `/jobs add` to put jobs in it before updating statuses." Do not show an empty table or ask which job to update.

If there are rows but none are active (all are Offer/Rejected/Withdrew), tell the user: "No active jobs to update. All entries are in terminal states (Offer/Rejected/Withdrew). Add a new job with `/jobs add` or run `/jobs find`."

---

Show all active jobs (Status not in `Offer` / `Rejected` / `Withdrew`) as a markdown table. Ask which job and what the new status.

---

## Suggested next actions per status

| New Status | Suggested Next Action |
|---|---|
| Applied | Follow up in 1 week |
| Recruiter Call | Prep: research company + refresh STAR stories |
| Phone Screen | Prep: technical review |
| Onsite | Prep: full interview prep + portfolio review |
| Offer | Negotiate by <date — ask user> |
| Rejected | Note reason if known |
| Withdrew | Note reason |

Ask: "Any notes to add? Next action?" Pre-fill the suggestion above but let the user override.

Update the row in `job_tracker.csv`. Confirm.

---

## Git commit

```
git commit -am "status: <Company> → <new status>"
```

(Skip if the user has uncommitted changes outside this row that they don't want bundled.)
