# OUTREACH (first DM to existing 1st-degree connections)

**Scope of this repo's outreach:** Send a first cold DM to people the user is **already connected to** (1st degree) at a target company. That's it.

**Out of scope (the user does these manually):**
- ❌ Sending connection requests
- ❌ Replying to DMs
- ❌ Sending follow-up nudges
- ❌ Sending DMs to 2nd-degree connections (they need a connection request first, which the user sends)

If the company is a `Connection Pending` row (no 1st-degree contact found yet), this flow logs candidates the user can connect with — but does NOT send the request.

---

## Step 1 — Pick a company

Show jobs where `Referral Needed=YES` and `Referral Status` is `Outreach Pending` or `Connection Pending`:

| # | Company | Role | Referral Status | Deadline | Days Left |

Days Left = Referral Deadline − today (negative = overdue).

Ask: "Which company?"

Direct argument shortcut: `/jobs outreach <Company>` jumps straight to that company.

---

## Step 2 — Find 1st-degree contacts at the company

**LinkedIn via Chrome:**

1. Navigate to `https://www.linkedin.com/company/<slug>/people/`. Extract the numeric company ID by scanning the page for `urn:li:fsd_company:<id>` and picking the most-frequent match.

2. Search **1st-degree only**:
   ```
   https://www.linkedin.com/search/results/people/?currentCompany=["<id>"]&network=["F"]
   ```
   No keyword filter. Read every result.

3. The URL params are `network` and `currentCompany`, NOT `facetNetwork` / `facetCurrentCompany`.

4. Extract per contact: name, title, profile URL, mutual-connection count.

**Result A: 1st-degree contacts found.** Rank them:
1. Recruiter / Talent Acquisition (highest priority for cold outreach)
2. Peer in similar role to the user (data scientist, engineer, etc. — match on title overlap with the user's profile)
3. Anyone else 1st degree at the company

Show ranked table:
```
# | Name | Title | LinkedIn URL | Mutuals
```

Ask: "Who do you want to reach out to? Pick one or more."

**Result B: No 1st-degree contacts found.** Optionally do a 2nd-degree search:

```
https://www.linkedin.com/search/results/people/?currentCompany=["<id>"]&network=["S"]&keywords=recruiter OR <role keyword>
```

Show top results with: `# | Name | Title | LinkedIn URL`. Tell the user:

> "No 1st-degree contacts at <Company>. Top 2nd-degree candidates above. **You'll need to send connection requests to them manually** — this repo doesn't send connection requests. Once any of them accept, run `/jobs outreach <Company>` again to send the first DM."

Log each as `1. Connection Pending` in `outreach/<Company>_contacts.md`. Update tracker `Referral Status=Connection Pending`. Stop here.

---

## Step 3 — Per selected 1st-degree contact: prior-thread check + DM

For each contact selected in Step 2:

### A. Open profile and verify connection level

Navigate to the contact's profile URL. Confirm they show as a 1st connection. If they're still 2nd → skip and log "Connection level mismatch" (the URL params can occasionally misclassify).

### B. Prior-thread check (MANDATORY before drafting)

Click Message to open the chat overlay. Scan the thread history:

| Thread state | Action |
|---|---|
| No prior messages | Proceed to **C (draft message)** |
| Prior message from user, contact replied | Skip — this is a warm thread, the user handles it manually |
| Prior message from user, no reply, ≥7 days ago | **SKIP** — do not re-engage. Log `Skipped (prior unanswered DM <date>)` |
| Prior message from user, no reply, <7 days ago | Skip for now (still in reply window) — log and move on |

Close the dialog if skipping.

### C. Draft and send the first DM

**Message style rules (strictly enforced):**
- Short: 2-3 sentences max
- Conversational, not corporate
- No em-dashes (—) anywhere
- No filler phrases like "I hope you're doing well"
- No "Best, <name>" sign-off — end naturally
- Use the user's pitch from `profile/profile.md` (third person for the background sentence)

**Draft template (adapt by contact's title):**

*If the contact is a recruiter:*

> Hi <First>, <user's third-person pitch from profile.md>. I saw the <Role> role at <Company> and it looks like a great fit. Would you be open to a referral or sharing more about the team?
>
> Thanks,
> <user's first name>

*If the contact is a peer (DS / engineer / similar):*

> Hi <First>, <user's third-person pitch from profile.md>. I saw the <Role> opening at <Company> and it looks like a great fit. Would you be open to a referral if it feels right? Would really appreciate it.
>
> Thanks,
> <user's first name>

Show the draft. Ask: "Send, edit, or skip?"

**On confirm:**
- Find the Message button → click → fill the message field with the draft.
- Ask: "Ready to send?"
- On "yes" → click Send.
- On "edit" → take the edit and re-show.
- On "skip" → close the dialog, log `Skipped (user choice)`, move on.

### D. Log to outreach file and tracker

`outreach/<Company>_contacts.md` (create if missing — see `_shared.md` for format):

```
| <Name> | <Title> | <LinkedIn URL> | 3. Cold DM Sent | <today> | First DM, 1st-degree |
```

Update tracker: `Referral Status=Outreach Sent`. Append to Notes: `Outreach: <Name> (<Title>) <today>`.

---

## Step 4 — After outreach

```
git commit -am "outreach: <Company> → <Name>"
```

Tell the user: "Done. Deadline: <Referral Deadline>. If they reply, handle the thread manually — this repo doesn't automate replies or follow-ups."

---

## Hard rules (re-stated)

- **Never click Connect** to send a connection request. If the only candidates are 2nd degree, log them as `Connection Pending` and tell the user to send requests manually.
- **Never click Send** without explicit user confirmation for that specific message.
- **Never use em-dashes** in the drafted message. Scan the draft for `—` and rewrite if present.
- **Don't ask the contact for their email or any other workaround.** This flow is text-only LinkedIn DMs to existing connections.
