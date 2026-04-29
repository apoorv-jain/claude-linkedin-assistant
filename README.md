<p align="center">
  <img src="assets/logo.png" width="600" alt="Claude LinkedIn Assistant" />
</p>

<p align="center">
  <strong>A Claude Code workspace for running a job search out of LinkedIn.</strong>
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

A Claude Code workspace for running a job search out of LinkedIn. Track jobs in a single CSV, discover new openings via LinkedIn + web search, send connection requests and first DMs at companies you're targeting.

> **Scope on purpose:** Claude only does the safe, repetitive parts. You handle anything that requires judgment (replies, applications, attachments, follow-ups).

## Quick Start

```bash
# 1. Clone
git clone https://github.com/FarzamHejaziK/claude-linkedin-assistant.git
cd claude-linkedin-assistant

# 2. Install the Claude in Chrome extension and sign into LinkedIn:
#    https://claude.com/claude-in-chrome

# 3. Drop your resume into the resumes/ folder. Any format works (.pdf, .tex, .md, .docx).
cp ~/Documents/your_resume.pdf resumes/

# 4. Open the repo in Claude Code
claude
```

Then in Claude Code:

```
/jobs                       # show the menu
/jobs daily                 # walk the full daily flow (find → check → outreach + commit)
/jobs find                  # discover new jobs and add to the tracker
/jobs check                 # daily dashboard
/jobs outreach <Company>    # send first DMs to your 1st-degree connections at <Company>
```

That's the whole loop. See [Requirements](#requirements) if any prereqs are missing, [What it does](#what-it-does) for the per-flow scope, and [Daily flow](#daily-flow) for what each step does.

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

## Requirements

1. **[Claude Code desktop app](https://docs.claude.com/en/docs/claude-code)** — the local desktop install that runs this workspace. The web version (claude.ai/code) won't work because it can't read your local `resumes/` folder or drive the Chrome extension.
2. **[Claude in Chrome extension](https://claude.com/claude-in-chrome)** — Claude needs this to navigate LinkedIn. Install it, then sign into your LinkedIn account in that browser.
3. **Git** (pre-installed on macOS / most Linux distros).
4. **GitHub CLI** (optional) — only if you want to push your fork.

See [REQUIREMENTS.md](REQUIREMENTS.md) for setup details.

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
