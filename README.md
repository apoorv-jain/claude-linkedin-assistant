# Claude LinkedIn Assistant

A Claude Code workspace for running a job search out of LinkedIn. Track jobs in a single CSV, discover new openings via LinkedIn + web search, and send a first outreach message to existing 1st-degree connections at companies you're targeting.

**Scope on purpose:** Claude only does the safe, repetitive parts. You handle anything that requires judgment (replies, applications, attachments, follow-ups).

## What it does

| Feature | Command | Who does what |
|---|---|---|
| Verify Chrome + LinkedIn login | (auto on every `/jobs` run) | Claude checks your active LinkedIn profile matches the one configured |
| Find new jobs | `/jobs find` | Claude searches LinkedIn + the web, scores results, adds them to the tracker |
| Track jobs | `/jobs check` · `/jobs add` · `/jobs update` | Claude reads/writes `job_tracker.csv` |
| Outreach (first message) | `/jobs outreach <Company>` | Claude drafts and sends a short DM to your **existing 1st-degree connections** at that company |
| Daily run | `/jobs daily` | Walks through find → check → outreach in one pass |

## What it does NOT do (by design)

- ❌ **No connection-request sending.** Outreach only DMs people you're already connected to.
- ❌ **No reply handling, no follow-ups.** When someone replies, you handle it.
- ❌ **No application submission, no resume tailoring, no cover letter generation.** Apply yourself.
- ❌ **No file uploads.** LinkedIn / Gmail / ATS forms block automated upload anyway.

If you want a more full-featured workflow (resume tailoring, reply handling, application form auto-fill), fork this repo and extend it. This version is intentionally minimal so it's safe to share.

## Requirements

1. **[Claude Code](https://docs.claude.com/en/docs/claude-code)** — the CLI that runs this workspace.
2. **[Claude in Chrome extension](https://claude.com/claude-in-chrome)** — Claude needs this to navigate LinkedIn. Install it, then sign into your LinkedIn account in that browser.
3. **Git** (pre-installed on macOS / most Linux distros).
4. **GitHub CLI** (optional) — only if you want to push your fork.

See [REQUIREMENTS.md](REQUIREMENTS.md) for setup details.

## Setup

```bash
# 1. Clone the repo
git clone https://github.com/<your-username>/claude-linkedin-assistant.git
cd claude-linkedin-assistant

# 2. Fill in your profile (used to verify the right LinkedIn account is active + write outreach messages)
cp profile/profile.template.md profile/profile.md
$EDITOR profile/profile.md

# 3. (Optional) Fill in personal info template if you want to reuse it later
cp profile/personal_info.template.json profile/personal_info.json
$EDITOR profile/personal_info.json

# 4. Open the repo in Claude Code
claude
```

Then in Claude Code:

```
/jobs
```

That runs the menu. Pick a sub-command, or jump straight in: `/jobs find`, `/jobs check`, `/jobs outreach Mixpanel`.

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
├── profile/
│   ├── profile.template.md       ← copy to profile.md, fill in
│   └── personal_info.template.json
│
├── outreach/                     ← created on first /jobs outreach run
│   └── <Company>_contacts.md     ← per-company contact log (gitignored by default)
│
└── .claude/
    ├── AUTOMATION_LIMITATIONS.md ← honest map of what's automated vs. manual
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

- `profile/personal_info.json` and `profile/profile.md` are `.gitignore`d by default.
- `outreach/*.md` is `.gitignore`d. Your contact log stays local.
- Only the templates ship in this repo.

If you fork to a private repo and want to commit your filled-in profile, edit `.gitignore`.

## Trust map

Read [.claude/AUTOMATION_LIMITATIONS.md](.claude/AUTOMATION_LIMITATIONS.md) before running anything end to end. Short version:

- **Safe to automate:** tracker reads/writes, job discovery, first DMs to existing connections.
- **You must do manually:** replies, follow-ups, applications, file uploads, anything with judgment.

## License

MIT. See [LICENSE](LICENSE).
