# OUTREACH — multi-platform startup outreach

Drafts personalized messages for each contact at a company across LinkedIn, Email, and Twitter/X. Shows ALL drafts to the user for review. **Sends nothing until the user explicitly approves each platform.**

---

## Step 1 — Pick a company

**If called with an argument** (`/startup-jobs outreach Stripe`): use that company.

**If no argument**: read `startup_tracker.csv`. Auto-pick the highest-priority company where:
- `Research Status=Done` AND
- `Referral Status=Outreach Pending` AND
- at least one contact has a reachable channel in `startup_outreach/<Company>_contacts.md`

Tiebreaker: soonest Referral Deadline. Print:
> `Selected: <Company> — <Role> (deadline: <date>)`

If no eligible company found, tell the user:
> "No companies ready for outreach. Run `/startup-jobs research` first to find contacts."

---

## Step 2 — Check research is done

Read `startup_outreach/<Company>_contacts.md`.

- File doesn't exist or is empty → tell the user: "No contacts found for <Company>. Run `/startup-jobs research <Company>` first." Stop.
- File exists → proceed.

---

## Step 2B — Detect cold outreach mode

Check `startup_tracker.csv` for this company's `Role` and `URL` columns:

- If `Role` is blank OR `Status=Exploring` → **cold outreach mode**
- If contacts file exists but `startup_tracker.csv` has no job URL → **cold outreach mode**

In cold outreach mode:
- Do not reference a specific role in any message.
- Use cold outreach templates from `_shared.md` ("Cold outreach — No open role").
- Print one line: `Cold mode: no open role found at <Company>. Using general-interest templates.`

In normal mode (role exists): use role-specific templates. Skip this section.

---

## Step 2C — Template check (cold mode only, skip if normal mode)

Check for a custom template in `resumes/outreach_templates.md`:

- File exists AND contains a template for `<Company>` or a general cold template → use it.
- File doesn't exist or has no matching template → prompt:

> "No custom cold outreach template found. I'll use the built-in general-interest template. Want to create a custom one for <Company> instead? (y/n)"

If `n`: use the built-in from `_shared.md` and proceed.

If `y`: run the **template creation flow**:

1. Ask:
   > "Tell me what you want to highlight — your specialization, why you're interested in <Company> specifically, or any team or product you're drawn to. One or two sentences is enough."

2. Ask:
   > "Any constraints? (e.g., keep under 100 words, reference a specific product launch, match a formal / casual tone)"

3. Draft a custom template using the user's input + the pitch built from their resume. Show the draft:
   ```
   [ Custom template draft ]
   Subject: <subject>

   Hi <First>,
   <body>

   Thanks,
   <user first name>
   ```

4. Ask:
   > "Use this just for <Company>, or save it as your reusable cold outreach template? (just this / save)"
   - `save` → write to `resumes/outreach_templates.md` (create if needed). Format:
     ```markdown
     # Cold Outreach Templates

     ## General cold template
     Subject: <subject>

     <body>
     ```
   - `just this` → use only for this run; do not write the file.

Proceed to Step 3 using whichever template was selected.

---

## Step 3 — Build the pitch from resume

Read the user's resume(s) in `resumes/`. Extract:
- **First name** (top of resume)
- **Current title + employer** (most recent role)
- **Years of experience** (from experience timeline)
- **Top 2-3 specializations** (headline / skills / repeated keywords)

Construct a single third-person pitch sentence. Example shape:
> "He is a senior software engineer with 6 years building fintech infrastructure, currently at <Employer>."

Rules:
- Third person for the background sentence; first person for greeting and sign-off.
- No em-dashes (—). No filler phrases ("I hope this finds you well").
- Never invent details not in the resume.
- Infer pronoun from name; if ambiguous, start with the title ("A senior software engineer with...").

---

## Step 3B — Dedup and follow-up check

Before drafting any messages, read the Outreach Log in `startup_outreach/<Company>_contacts.md`.

**Date arithmetic:** parse every log date as YYYY-MM-DD. Compute `days_since = today - log_date` in whole calendar days. Today's date comes from the `currentDate` context variable.

For each contact + platform, find the most recent log row and determine the action:

| Most recent log status | days_since | Action |
|---|---|---|
| No entry | — | Draft initial outreach |
| `Draft Loaded` or `Draft Loaded in Gmail` | any | **Ask first:** "Did you send the draft loaded on <log_date>? (y/n/skip)" — update status before deciding |
| `Approved — Sent` | < 7 | Skip — too soon. Print: "Contacted <days_since>d ago, skip until <log_date + 7d>." |
| `Approved — Sent` | >= 7 | Draft follow-up (not initial) |
| `Followed Up` | < 14 | Skip — already followed up <days_since>d ago. |
| `Followed Up` | >= 14 | Draft second follow-up only if user confirms |
| `Connection Pending` | any | Skip LinkedIn DM — connection not yet accepted. |
| `Replied` / `Declined` / `Converted` | any | Skip — conversation already resolved. |
| `User Skipped — not sent` | any | Treat as No entry — draft initial outreach. |

**If a `Draft Loaded` entry is found:** stop and ask the user to clarify before drafting anything new:
> "I see a draft was loaded for <Name> via <platform> on <log_date> but not marked sent. Did you send it?"
> - `yes` → update status to `Approved — Sent`, set date to <log_date>. Then check the 7-day rule.
> - `no` → update status to `User Skipped — not sent`. Draft initial outreach fresh.
> - `skip` → leave status as-is, skip this contact for today.

If ALL platforms for ALL contacts are skipped, tell the user and stop.

If a contact needs a follow-up (Approved — Sent >= 7 days, no newer entry): use the follow-up templates from `_shared.md` instead of the initial templates.

Apply platform guardrails from `_shared.md` before drafting. Note any guardrail blocks in the draft preview.

---

## Step 4 — Template selection

**First: check for saved templates.** Before showing any options, look for `resumes/outreach_templates.md`.

- **File does not exist** → prompt immediately:
  > "No outreach templates saved yet. Would you like to create a custom template for this outreach, or use the built-in defaults?
  >
  >   `create` — I'll guide you through writing one (takes ~1 min)
  >   `built-in` — use the auto-matched defaults and continue"

  - `create` → jump to the template creation flow (option `3` below), then return here.
  - `built-in` → proceed with built-in templates. Skip the rest of Step 4 setup.

- **File exists** → read it, note how many templates are saved, and continue to the selection prompt below.

---

Before drafting anything, show the user the template plan and ask them to choose.

Print a table of contacts and their auto-assigned template:

```
Template plan for <Company>:

  Contact              | Type       | Template
  ---------------------|------------|---------------------------
  Jane Smith           | Recruiter  | Built-in Recruiter
  Bob Lee              | HM         | Built-in HM/EM
  Alice Chen           | CEO        | Built-in CEO (<100 emp)
```

Then ask:

> Which template approach would you like?
>
>   `1` — Use built-in templates as shown above (auto-matched by contact type)
>   `2` — Use a saved custom template from `resumes/outreach_templates.md`
>   `3` — Write a new template now (I'll help you draft it and offer to save it)
>   `4` — Different template per contact (I'll ask for each one)
>
> Press Enter to use built-in defaults.

**If `1` or Enter:** proceed with built-in templates. Skip to Step 4A.

**If `2`:** check `resumes/outreach_templates.md`. If it exists, read it and show available templates by name. Ask which one to apply. If the file doesn't exist, tell the user:
> "No saved templates found. Want to create one now? (y/n)"
> - `y` → run option `3` flow below.
> - `n` → fall back to built-in templates.

**If `3`:** run the template creation flow:
1. Ask: "What do you want to highlight — your specialization, why you're interested in <Company>, or a specific team or product? One or two sentences."
2. Ask: "Any constraints? (e.g., keep under 100 words, casual tone, reference a specific product)"
3. Draft a custom template using their input + the pitch from Step 3. Show it:
   ```
   [ Draft custom template ]
   LinkedIn DM:
   Hi <First>, ...

   Email subject: ...
   Email body:
   Hi <First>, ...

   Twitter/X:
   Hi <First>, ...
   ```
4. Ask: "Use this for <Company> only, or save it as a reusable template? (just this / save)"
   - `save` → write to `resumes/outreach_templates.md` (create if needed). Add under a heading matching the contact type or `## Custom — <Company>`.
   - `just this` → use for this run only.

**If `4`:** after drafting each contact's message in Step 4A, pause and ask:
> "Template for <Name> (<Title>): use built-in, custom, or write new? (1/2/3)"
Process per-contact before moving to the next.

---

## Step 4A — Draft messages for each contact + platform

For each contact in the file, draft one message per platform where that contact has a confirmed or inferred channel. Use the template selected in Step 4.

**If cold outreach mode** (Step 2B): use the "Cold outreach — No open role" templates from `_shared.md` (or the custom template from Step 2C). Skip the Recruiter / HM / CEO role-specific templates below — those require an open role to reference.

### Platform: LinkedIn DM

Check prior-thread rule first (same as `jobs/outreach.md` Step 3B): if there is already an unanswered message sent <7 days ago, skip. If ≥7 days unanswered, skip and log.

**Template — Recruiter:**
> Hi <First>, <third-person pitch>. I came across the <Role> opening at <Company> and it looks like a great fit. Would you be open to a referral or sharing more about the team?
>
> Thanks,
> <user's first name>

**Template — Engineering Manager / Hiring Manager:**
> Hi <First>, <third-person pitch>. I noticed the <Role> role at <Company> and it aligns well with what I've been building. Would you be open to a quick chat or a referral if it feels right?
>
> Thanks,
> <user's first name>

**Template — CEO (companies < 100 employees only):**
> Hi <First>, <third-person pitch>. I've been following <Company>'s work on <brief 1-sentence description of what the company does, inferred from research>. I saw you're hiring for <Role> and would love to explore if there's a fit.
>
> Thanks,
> <user's first name>

### Platform: Email

Subject line options (show two, let user pick):
- `<Role> at <Company> — <user's first name>`
- `Interested in <Role> — <user's current title>`

Body (same tone rules — no em-dashes, concise, third person pitch):

> Hi <First>,
>
> <third-person pitch>. I came across the <Role> opening at <Company> and it aligns closely with the work I've been doing in <top specialization>.
>
> I'd love to connect briefly or learn about the team if you're open to it.
>
> Thanks,
> <user's first name>
> <user's LinkedIn URL, pulled from resume or search_profile.md if present>

If the email confidence is `Guessed` or `Pattern-inferred`, add a note in the draft preview to the user:
> ⚠ Email confidence: pattern-inferred (`{first}@company.com`). Verify before sending.

### Platform: Twitter/X DM

Keep under 280 characters. No subject line.

> Hi <First>, <one-sentence third-person pitch>. I saw <Company> is hiring for <Role> — would love to connect if you're open to it. Thanks, <user first name>

---

## Step 5 — Present ALL drafts for approval

**DO NOT send anything yet.**

**Timing recommendation** (from `_shared.md` timing windows): print one line before the drafts:

> Best time to send: Tue-Thu, 9-11am or 4-6pm recipient's timezone (Email); 8-10am or 5-7pm (LinkedIn); 10am-12pm (Twitter/X).

If today is Saturday or Sunday: add:
> Today is <day>. Consider staging these and sending Tuesday morning for better open rates.

If today is Friday: add:
> Today is Friday. Friday sends typically get lower response rates — consider sending Tuesday instead.

---


Print every draft in a clearly separated block:

```
============================================================
CONTACT 1: Jane Smith (CEO) — <Company>
============================================================

[ LinkedIn DM ]
---
Hi Jane, ...
---

[ Email ]
Subject: Senior Engineer at Stripe — Alex
---
Hi Jane, ...
---
To: jane@stripe.com  (confidence: confirmed)

[ Twitter/X DM ]
---
Hi Jane, <pitch>. ...
---
@janesmith

============================================================
CONTACT 2: Bob Lee (VP Engineering) — <Company>
============================================================
...
```

Then ask:

> Review the drafts above. For each contact, tell me which platforms to send on.
>
> Options:
>   - `all` — send on all platforms shown
>   - `linkedin` / `email` / `twitter` — send on specific platform(s) only
>   - `skip <name>` — skip this contact entirely
>   - `edit <contact> <platform>` — paste your revised message and I'll use that instead
>
> Format: `Jane: email, linkedin | Bob: skip | Alice: all`
>
> Or type `cancel` to exit without sending anything.

**Wait for the user's response before proceeding. Do not send anything.**

---

## Step 6 — Load compose windows (do NOT send)

Parse the user's approval response. For each approved contact + platform, load the compose window and stop. The user clicks Send.

### LinkedIn DM

1. Navigate Chrome to the contact's LinkedIn profile URL.
2. Confirm connection degree is 1st. If 2nd or 3rd, skip DM — send a connection request ("Send without a note") instead, log as `Connection Pending`, and tell the user.
3. Click Message. Paste the approved draft into the compose field.
4. **Stop. Do not click Send.** Tell the user: "LinkedIn DM ready for <Name>. Click Send when you're ready."
5. Log in `startup_outreach/<Company>_contacts.md`: `LinkedIn | Draft Loaded | <today>`

### Email

1. Navigate Chrome to `https://mail.google.com/mail/u/0/#compose`.
2. Fill in: To = `<email>`, Subject = `<chosen subject>`, Body = `<approved draft>`.
3. **Stop. Do not click Send.** Tell the user: "Email ready for <Name> in Gmail. Click Send when you're ready."
4. Log in contact file: `Email | Draft Loaded in Gmail | <today>`

### Twitter/X DM

1. Navigate Chrome to `https://twitter.com/messages/compose?recipient_id=<handle>` (or search the handle if numeric ID unknown).
2. Paste the approved draft into the message field.
3. **Stop. Do not click Send.** Tell the user: "Twitter/X DM ready for <Name>. Click Send when you're ready."
4. Log in contact file: `Twitter | Draft Loaded | <today>`

**Stop condition:** if Chrome shows a CAPTCHA or rate-limit at any point, stop immediately. Report which windows loaded and which are still pending.

---

## Step 7 — Confirm what was sent

After all compose windows are loaded, pause and ask the user:

> All drafts are loaded in their compose windows. Go ahead and click Send in each tab.
>
> Once done, come back and tell me what went through so I can update the log:
>
>   `all sent` — everything was sent as loaded
>   `Jane: linkedin, email | Bob: skipped | Alice: twitter` — per-contact breakdown
>   `none` — nothing sent yet, remind me later
>
> Marking accurately matters: the follow-up scheduler uses these dates.

**Wait for the user's response. Do not update any status to `Approved — Sent` until the user confirms.**

Parse the response and update `startup_outreach/<Company>_contacts.md`:

- Contact confirmed sent → change log status from `Draft Loaded` → `Approved — Sent`. Set date to today.
- Contact skipped → change status to `User Skipped — not sent`.
- `none` / `remind me later` → leave all statuses as `Draft Loaded`. Add a reminder line to the Step 8 summary.

Update `startup_tracker.csv` only if at least one message was confirmed sent by the user:
- `Referral Status=Outreach Sent`
- Append to `Notes`: `Outreach: <Name> (<Title>) via <platforms> <today>`

For follow-up sends: add a new log row with status `Followed Up` (do not overwrite the original row).
For contacts skipped by guardrails or dedup: log the reason in the Notes column.

---

## Step 8 — Summary

Print:

```
Outreach complete — <Company>

  Jane Smith: LinkedIn sent · Email sent · Twitter — not confirmed
  Bob Lee: skipped by user
  Alice Chen: Draft Loaded — awaiting confirmation

Confirmed sent: <N> | Loaded, unconfirmed: <N> | Skipped: <N>
```

If any drafts are unconfirmed, add:
> "Unconfirmed drafts are logged as 'Draft Loaded'. Run `/startup-jobs check` to see follow-up reminders once you've sent them and marked them."

Then:
```
git commit -am "startup-jobs outreach: <Company> — <N> confirmed sent, <N> loaded"
```

---

## Hard rules

- **Never click Send on any platform — ever.** Load the compose window, paste the content, stop. LinkedIn, Gmail, Twitter — all three. The user clicks Send, not Claude.
- **Never mark a message `Approved — Sent` until the user explicitly confirms they clicked Send** (Step 7). Draft Loaded is not Sent.
- **All drafts shown before any window is opened.** Step 5 shows everything; Step 6 only runs after the user's approval response.
- **No em-dashes (—) in any draft.** Scan every draft before displaying. Rewrite if found.
- **No file attachments.** Outreach is text-only. If the user wants to attach a resume, they do it manually after the compose window is open.
- **Email to guessed addresses:** warn the user clearly with the confidence label. Never silently load a guessed email address into the To field without a ⚠ warning.
- **CEO outreach only for companies < 100 employees.** For larger companies, skip the CEO and focus on Engineering Manager or Recruiter.
- **Draft Loaded entries must be resolved before scheduling follow-ups.** Always ask "did you send this?" for any unconfirmed draft before computing the 7-day follow-up window.
