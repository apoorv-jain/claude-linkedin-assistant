# UPDATE (status change)

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
