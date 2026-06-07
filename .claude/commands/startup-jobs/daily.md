# DAILY — startup-jobs orchestrator

Walks through the full daily startup-jobs pipeline end to end with a checkpoint between each step. Designed to be run once per day.

---

## Sequence

```
1. check      — dashboard snapshot + daily checklist
2. find       — discover new roles at VC-backed companies
3. research   — deep contact discovery for HIGH priority companies
4. outreach   — draft + review (Claude loads compose windows; you click Send)
4b. mark      — confirm what you sent; Claude updates the log
5. commit     — single git commit summarizing the day
```

Each step has a checkpoint. The user can continue, skip, or quit.

---

## Step 1 — Check (dashboard)

Read and execute `check.md` in full.

After output, ask:
> Continue to find new jobs? (y / skip / quit)

- `y` → Step 2
- `skip` → Step 3
- `quit` → go to Step 5 (commit whatever changed today)

---

## Step 2 — Find

Read and execute `find.md` in full.

Use filters already resolved from search_profile.md and resume — do not re-prompt for them. If the user wants different filters, they can run `/startup-jobs find --fund yc --type Remote` separately.

After output, ask:
> Continue to research contacts at HIGH priority companies? (y / skip / quit)

- `y` → Step 3
- `skip` → Step 4
- `quit` → Step 5

---

## Step 3 — Research

Read and execute `research.md` for each HIGH priority company in `startup_tracker.csv` where `Research Status=Not Started`, up to a maximum of 3 companies per daily run (to avoid a very long session).

If there are more than 3 eligible companies, tell the user:
> "Found <N> companies needing research. Running research for the top 3 by deadline. Re-run `/startup-jobs daily` tomorrow to cover the rest."

After each company's research completes, ask:
> Research done for <Company>. Continue to next company? (y / skip all / quit)

After all research is done (or skipped), ask:
> Continue to outreach? (y / skip / quit)

---

## Step 4 — Outreach

Read and execute `outreach.md` for each company where:
- `Research Status=Done` AND
- `Referral Status=Outreach Pending`

Process one company at a time. The outreach flow will:
1. Show all drafts for review (Step 5 in outreach.md)
2. Load approved drafts into compose windows (Step 6 — Claude stops, you click Send)
3. Ask you to confirm what was sent (Step 7 — required before log is updated)

After each company's confirm step:

> Outreach loaded for <Company>. Sent everything? Continue to next company? (y / quit)

**After all companies:** print a consolidated TODO for the user:

```
TODO — mark as sent
  Return here after clicking Send in each tab and tell me:
    `all sent` — or — `<Company>: Jane linkedin, Bob skipped | <Company2>: all`
  I'll update the log and set follow-up dates from today.
```

If the user marks immediately, process it now. If they say `later`, leave all statuses as `Draft Loaded` and add this TODO to the Step 5 commit message.

---

## Step 5 — Commit

Summarize what happened in this daily run:

```
Today's run:
  - Found: <N> new roles added to startup_tracker.csv
  - Researched: <companies>
  - Outreach drafts loaded: <N> contacts across <companies>, platforms: <list>
  - Confirmed sent: <N> | Pending confirmation: <N>
```

If any drafts are unconfirmed:
```
ACTION NEEDED:
  [ ] Click Send in open compose tabs, then run `/startup-jobs outreach` and
      confirm what went through so follow-up dates are set correctly.
```

Then:
```
git add startup_tracker.csv startup_outreach/
git commit -m "startup-jobs daily: found <N> roles, researched <companies>, outreach <N> loaded (<N> confirmed sent)"
```
