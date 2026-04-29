<p align="center">
  <img src="assets/logo.png" width="600" alt="Claude LinkedIn Assistant" />
</p>

<p align="center">
  <strong>A Claude Code assistant for LinkedIn.</strong>
</p>

<p align="center">
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-green.svg"></a>
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude%20Code-Ready-orange">
  <img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-via%20Chrome%20MCP-0A66C2?logo=linkedin&logoColor=white">
</p>

<p align="center">
  <a href="#quick-start"><strong>Quick Start</strong></a> ·
  <a href="#what-it-does"><strong>What it does</strong></a> ·
  <a href="#daily-flow"><strong>Daily flow</strong></a> ·
  <a href="#privacy"><strong>Privacy</strong></a>
</p>

A Claude Code assistant for LinkedIn. Track jobs in a single CSV, discover new openings via LinkedIn + web search, send connection requests and first DMs at companies you're targeting.

> **Scope on purpose:** Claude only does the safe, repetitive parts. You handle anything that requires judgment (replies, applications, attachments, follow-ups).

## Quick Start

Before anything else, you'll want three things installed:

- **[Claude Code desktop app](https://docs.claude.com/en/docs/claude-code)** — this is the app where you'll actually run everything. It has to be the desktop app, not the web version at claude.ai/code. The web version can't see your local files (resume, tracker CSV, contact log) and can't drive the Chrome extension, so it just won't work for this repo.
- **[Claude in Chrome extension](https://claude.com/claude-in-chrome)** — this is how Claude drives LinkedIn for you. Install it in Chrome, sign in to your LinkedIn account in that same browser, and keep the window open whenever you run a `/jobs` flow. Without this extension, none of the LinkedIn automation works.
- **Git** — already on your Mac. You'll only use it once, in step 1 below.

Once you've got those, here's the flow.

### 1. Clone the repo

Open your terminal and run:

```bash
git clone https://github.com/FarzamHejaziK/claude-linkedin-assistant.git
cd claude-linkedin-assistant
```

That's it for the terminal. Everything else happens inside the Claude Code app.

### 2. Put your resume into `resumes/`

There's no profile file or settings form to fill out. Claude reads your resume directly to figure out your name (so it can verify the right LinkedIn account is signed in), your target roles + top skills (for `/jobs find` searches), and a short pitch (for the outreach DM template).

Just put your resume into the `resumes/` folder however you normally move a file — drag it in from Finder, copy it from wherever you keep your latest version, whatever's easiest. `.pdf`, `.tex`, `.md`, and `.docx` all work. If you have role-specific versions of your resume, drop them all in — the flows read everything in there and use the union of titles + skills.

### 3. Open the folder in Claude Code

Launch the **Claude Code desktop app**, then **File → Open Folder** and pick the `claude-linkedin-assistant` folder you just cloned. The app loads the workspace and finds all the `/jobs` commands automatically.

### 4. Run `/jobs`

In the chat, type `/jobs` and hit enter. The first thing it does, every time, is verify that the Chrome extension is connected and the LinkedIn account signed in matches the name on your resume. If something's off, it tells you exactly what to fix instead of guessing.

After that, you've got the menu:

```
/jobs                       # show the menu (start here the first time)
/jobs daily                 # walk the full daily flow end-to-end
/jobs find                  # discover new jobs and add them to the tracker
/jobs check                 # daily dashboard: what's pending, what's stale
/jobs outreach <Company>    # connection requests + first DMs at <Company>
/jobs add                   # manually add a job you found somewhere else
/jobs update                # change a job's status (Phone Screen, Onsite, Rejected)
```

If you just want to see it run, do `/jobs daily` — it walks you through one full day end to end.

For more detail on any of these, see [What it does](#what-it-does), [Daily flow](#daily-flow), or [REQUIREMENTS.md](REQUIREMENTS.md).

## What it does

| Feature | Command | Who does what |
|---|---|---|
| Verify Chrome + LinkedIn login | (auto on every `/jobs` run) | Claude reads your name from your resume in `resumes/` and verifies the active LinkedIn profile matches |
| Find new jobs | `/jobs find` | Claude reads your resume(s), searches LinkedIn + the web, scores and adds results to the tracker |
| Track jobs | `/jobs check` · `/jobs add` · `/jobs update` | Claude reads/writes `job_tracker.csv` |
| Outreach (cold sweep) | `/jobs outreach <Company>` | Claude sends connection requests to 2nd-degree contacts ("Send without a note") and a first DM to existing 1st-degree connections at that company |
| Daily run | `/jobs daily` | Walks through find → check → outreach in one pass |

## What it does NOT do (by design)

- ❌ **No reply handling, no follow-ups.** When someone replies to a DM, you handle the conversation.
- ❌ **No application submission, no resume tailoring, no cover letter generation.** Apply yourself.
- ❌ **No file uploads.** LinkedIn / Gmail / ATS forms block automated upload anyway.
- ❌ **No personalized connection-request notes.** All connection requests go through "Send without a note" to conserve LinkedIn's monthly personalized-invite quota.

If you want a more full-featured workflow (resume tailoring, reply handling, application form auto-fill), fork this repo and extend it. This version is intentionally minimal so it's safe to share.

## Daily flow

`/jobs daily` walks through the recommended sequence end to end:

```
1. /jobs check       — dashboard: what's pending, what's stale
2. /jobs find        — discover new jobs via LinkedIn + web search
3. /jobs add         — capture interesting finds (auto-flags companies that need a referral)
4. /jobs outreach    — send first DMs to 1st-degree connections at companies that need outreach
5. commit            — single git commit at the end
```

You can also run sub-commands individually any time.

## Folder layout

```
claude-linkedin-assistant/
├── README.md                     ← this file
├── REQUIREMENTS.md               ← setup steps
├── CLAUDE.md                     ← project rules loaded by Claude
├── LICENSE                       ← MIT
├── .gitignore
├── job_tracker.csv               ← MAIN DASHBOARD. Open in Numbers/Excel/VS Code.
│
├── resumes/                      ← drop your resume(s) here (gitignored by default)
│   └── README.md                 ← committed instructions
│
├── outreach/                     ← created on first /jobs outreach run
│   └── <Company>_contacts.md     ← per-company contact log (gitignored by default)
│
└── .claude/
    └── commands/
        ├── jobs.md               ← top-level /jobs command
        └── jobs/
            ├── _shared.md        ← rules loaded by every sub-flow
            ├── find.md           ← discover jobs
            ├── check.md          ← daily dashboard
            ├── add.md            ← add a job to the tracker
            ├── update.md         ← change status
            ├── outreach.md       ← first-message DM flow
            └── daily.md          ← orchestrator
```

## Tracker format

`job_tracker.csv` columns:

```
Priority, Company, Role, Location, Type, Salary, Status, Applied Date,
Next Action, URL, Notes, Discovered Date, Referral Needed, Referral Status,
Referral Deadline, Apply Via
```

Canonical statuses: `To Apply` → `Applied` → `Recruiter Call` → `Phone Screen` → `Onsite` → `Offer` / `Rejected` / `Withdrew`

Open in any spreadsheet app or use the `/jobs check` dashboard.

## Privacy

- `resumes/*` (everything except `resumes/README.md`) is `.gitignore`d by default. Your resume stays local.
- `outreach/*.md` is `.gitignore`d. Your contact log stays local.

If you fork to a private repo and want to commit your resume too, edit `.gitignore`.

## License

MIT. See [LICENSE](LICENSE).
