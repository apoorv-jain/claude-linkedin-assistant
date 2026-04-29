# CLAUDE.md — Claude LinkedIn Assistant

Project rules loaded automatically when Claude works in this folder.

## What this is

A minimal job-search workspace that pairs `job_tracker.csv` with four Claude-driven flows: find, check/add/update tracker rows, and send a first outreach DM to existing 1st-degree LinkedIn connections.

See `README.md` for the user-facing overview and `REQUIREMENTS.md` for setup. The authoritative command flows live in `.claude/commands/jobs/`.

## Where the docs live (read before acting)

| File | Purpose |
|---|---|
| `CLAUDE.md` (this file) | Project-wide hard rules, canonical values |
| `README.md` | Human-readable workflow overview |
| `REQUIREMENTS.md` | Setup (Chrome extension, git) |
| `.claude/AUTOMATION_LIMITATIONS.md` | Trust map: what Claude automates vs. what the user must do |
| `.claude/commands/jobs/_shared.md` | Shared rules loaded by every `/jobs` sub-flow |
| `.claude/commands/jobs/{find,check,add,update,outreach,daily}.md` | Per-flow procedures |
| `profile/profile.md` | User-provided profile: name, target roles, pitch (loaded by find + outreach) |
| `profile/personal_info.json` | (Optional) structured personal info; not used by any default flow |

## User profile

The user's profile lives in `profile/profile.md` (created by them from `profile.template.md`). Read it whenever you need:
- The user's LinkedIn name (to verify the right account is active in Chrome)
- Target roles + skills (for `/jobs find`)
- Short pitch (for `/jobs outreach` message drafting)

If `profile/profile.md` doesn't exist, tell the user to copy the template and fill it in before running any flow.

## Hard rules

1. **`job_tracker.csv` is the single source of truth.** When updating, preserve every untouched row exactly.
2. **URL column is mandatory.** Every new job row must have a working apply link. Ask or search before saving.
3. **Application folders are dated** if created: `applications/<YYYY-MM-DD>_<Company>_<Role>/`. Sanitize names (spaces → underscores, strip commas). This repo doesn't ship application infra by default; create the folder only if the user explicitly asks.
4. **Date format:** always `YYYY-MM-DD`.
5. **Render tracker contents as a clean markdown table** — never raw CSV.
6. **NEVER use em-dashes (—) in any user-facing message the user sends** — emails, LinkedIn DMs, follow-ups, subject lines, anything outgoing. Use commas, periods, parentheses, or split sentences instead. Em-dashes are a tell of AI-written text. Scan every draft for `—` before showing it; rewrite if found. Internal notes, tracker, contacts files are fine.
7. **For outreach DMs: you draft the message, the user sends it.** Browser MCP can navigate to the thread and type the message body, but you do NOT click Send without explicit confirmation. Standing flow: (a) navigate to the thread, (b) type the message body, (c) STOP and ask: "Ready to send?", (d) wait for the user to confirm before clicking Send.
8. **Never send connection requests.** This repo's outreach flow is **first message only, to existing 1st-degree connections**. If the target is a 2nd-degree connection, log them as "Connection Pending" so the user can send the request manually, but do NOT click Connect.
9. **Never attempt file uploads.** LinkedIn / Gmail / ATS file inputs are blocked from automation. If a flow requires an attachment, type the message and stop — the user handles the attach + send.

## Canonical values

- **Statuses:** `To Apply` → `Applied` → `Recruiter Call` → `Phone Screen` → `Onsite` → `Offer` / `Rejected` / `Withdrew`
- **Priorities:** `HIGH`, `MEDIUM`, `LOW`
- **Referral Status:** `Not Needed` / `Outreach Pending` / `Connection Pending` / `Outreach Sent` / `Got Referral` / `Declined` / `No Referral`

## Referral-needed threshold

Companies with **>500 employees OR >100K LinkedIn followers** → `Referral Needed=YES`. Otherwise NO.

When `Referral Needed=YES`:
- `Referral Deadline = Discovered Date + 5 days`
- Outreach via LinkedIn DM (Chrome) to existing 1st-degree connections at the company.
- If no 1st-degree connection exists, surface that as a "Connection Pending" candidate the user must connect with manually.

## Unified command

Use `/jobs` for all tracker operations. Sub-flows: `daily` · `check` · `find` · `add` · `update` · `outreach`

## Tone

Match the user's voice. Skip over-explanation. Default to terse, technical responses. When drafting outreach DMs, follow the templates in `outreach.md` and the user's pitch in `profile/profile.md` — do not invent claims about their experience.
