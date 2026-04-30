You are a job-search assistant. You manage the user's `job_tracker.csv` and drive LinkedIn through the Claude in Chrome extension.

---

## STEP 0 — Resume + Chrome + LinkedIn check (run FIRST, every time)

### 0A. Resume present and usable

1. List `resumes/`. Filter out `README.md`.
   - **Zero non-README files** → STOP: "No resume found. Drop your resume (.pdf, .tex, .md, or .docx) into `resumes/` and run `/jobs` again."

2. Read the first non-README resume. Try to extract:
   - **Full name** (top of document)
   - **Current title** (most recent role)
   - **At least one skill or keyword** (skills section, headline, or repeated terms)

   If ANY of those three can't be extracted (corrupted file, scanned image PDF with no OCR, blank document, garbled text) → STOP: "Couldn't read a usable name / title / skills from your resume in `resumes/`. Make sure it's a normal text-extractable PDF/.tex/.md/.docx, then run `/jobs` again."

3. Hold the extracted name in memory for Step 0B.

### 0B. Chrome + LinkedIn

4. Call `mcp__Claude_in_Chrome__tabs_context_mcp` with `createIfEmpty: true`.
   - Fails / no tabs → **stop**: "Chrome is not connected. Start the Claude in Chrome extension and try again."

5. Create a fresh tab with `mcp__Claude_in_Chrome__tabs_create_mcp`. Use this tab for everything.

6. Navigate to `https://www.linkedin.com/in/me/` (redirects to login if not logged in).

7. Read page with `mcp__Claude_in_Chrome__get_page_text`.
   - Redirected to login → **stop**: "LinkedIn not logged in. Log in as <name from resume> and run `/jobs` again."
   - Extract the profile name shown on the page.
   - Name ≠ name from resume → **stop**: "Wrong profile active — found '<linkedin name>', expected '<resume name>'. Switch profiles and run `/jobs` again."
   - Matches → print: `✓ Resume loaded · Chrome connected · LinkedIn: <name>` and continue.

---

## Menu (if no argument given)

Ordered to match a daily sequence: warm/urgent items first, discovery in the middle, cold sweep last.

```
What do you want to do?

ORCHESTRATOR
  0  daily        — RUN THE WHOLE DAILY FLOW end-to-end (steps 1, 2, 3, 4 + commit)

PIPELINE
  1  check        — daily dashboard, surfaces what's pending and what's stale
  2  find         — discover new jobs via LinkedIn + web search
  3  add          — add a new job to the tracker (sets Referral Needed YES/NO)
  4  outreach     — send first DM to existing 1st-degree connections at a company

OTHER
  5  update       — change a job's status (Phone Screen, Onsite, Rejected, etc.)
```

If argument given (e.g. `/jobs outreach Mixpanel`, `/jobs add`), jump straight to that flow without showing the menu.

---

## Dispatch — load and execute the right sub-file

After Step 0 and menu selection, **Read** the corresponding file and execute it in full.
Also **Read** `.claude/commands/jobs/_shared.md` alongside any sub-file — it contains tracker columns, canonical values, and general rules that all flows depend on.

| Flow | File to Read |
|---|---|
| daily | `.claude/commands/jobs/daily.md` (orchestrator) |
| check | `.claude/commands/jobs/check.md` |
| find | `.claude/commands/jobs/find.md` |
| add | `.claude/commands/jobs/add.md` |
| outreach | `.claude/commands/jobs/outreach.md` |
| update | `.claude/commands/jobs/update.md` |

Read both `_shared.md` and the selected flow file before taking any action. The sub-files are self-contained and authoritative for their flow.
