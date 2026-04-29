# DAILY (orchestrator)

`/jobs daily` runs the daily sequence end to end. Each step has a checkpoint: the user can run, skip, or stop. Empty steps are auto-skipped.

For ad-hoc work, run individual `/jobs <sub-command>` directly.

---

## How this command behaves

1. Run STEP 0 (Chrome + LinkedIn check from `jobs.md`) once at the start.
2. Walk through the steps below in order.
3. Before each step, print a one-line preview of what's queued. If the queue is empty, auto-skip and announce.
4. Between steps, ask: `[Enter] continue / [s] skip / [q] quit`. Proceed on continue, move to next on skip, exit on quit.
5. At the end, run the commit step automatically and print the daily summary.

If the user invokes a single sub-command from inside the daily flow (e.g. types `/jobs outreach Walmart`), exit the daily orchestrator and let that sub-command run standalone.

---

## Step 1 — `/jobs check` (dashboard)

Read `check.md`, execute. Show all sections.

Then ask: continue / skip / quit?

---

## Step 2 — `/jobs find` (discover new jobs)

Read `find.md`. Surface candidate jobs based on the user's profile.

Then ask: continue / skip / quit?

---

## Step 3 — `/jobs add` (capture finds)

For each job surfaced in Step 2, read `add.md` and add to the tracker. The find flow already auto-adds in batch — this step exists to handle anything the user wants to add manually that wasn't surfaced (e.g. a link they saw elsewhere).

Skip if there's nothing manual to add.

Then ask: continue / skip / quit?

---

## Step 4 — `/jobs outreach` (cold sweep on new HIGH companies that need a referral)

**Skip if:** no `Referral Status=Outreach Pending` companies with `Priority=HIGH`.

**Else:** for each new HIGH company that needs a referral, read `outreach.md` and run the sweep. The sweep does both: connection requests to 2nd-degree contacts ("Send without a note", per-company quota from Step 2C) AND first DMs to existing 1st-degree connections. Keep going across companies until LinkedIn pushes back (CAPTCHA / rate-limit notice) — at which point stop and tell the user.

Then ask: ready to commit / skip commit / quit?

---

## Step 5 — Commit

Run a single git commit covering everything done in this session. Group changes by category in the commit message. Format:

```
git add -A && git commit -m "$(cat <<'EOF'
daily <YYYY-MM-DD>: <short summary>

New jobs added:
- <Company>: <Role> [<Priority>]

Connection requests sent:
- <Company>: <N> contacts (no-note)

Outreach DMs sent:
- <Company>: <Name>

Status updates:
- <Company>: <old> → <new>

EOF
)"
```

Skip if no changes were made.

---

## Final summary

After commit (or after the user quits), print:

```
Daily run complete (<YYYY-MM-DD>):
  ✅ <I> new jobs added
  ✅ <K> connection requests sent (no-note) — total in last 7 days: <X>
  ✅ <M> first DMs sent to 1st-degree connections
  ⏳ Pending: <list of items waiting on the user>

Tomorrow's surface:
  - <Co> deadline tomorrow
  - <Co> awaiting acceptance for <N> connection requests
```

---

## Hard rules (re-stated)

- Connection requests always use "Send without a note". Per-company quota from `outreach.md` Step 2C; no global weekly cap (loop only stops on LinkedIn-side block per Step 4B).
- Never sends a DM with an attachment. The first DM is text only.
- Never click Send on a DM without explicit user confirmation per message. Connection-request batches use a single batch confirmation.
- Never use em-dashes in any drafted message.
- One commit at the end of the daily run, not per-step.
