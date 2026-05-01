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
| `.claude/commands/jobs/_shared.md` | Shared rules loaded by every `/jobs` sub-flow |
| `.claude/commands/jobs/{find,check,add,update,outreach,daily}.md` | Per-flow procedures |
| `resumes/` | User's resume(s). Source of truth for name, target roles, skills, pitch |
| `resumes/search_profile.md` | (Optional) free-form preferences: must-haves, deal-breakers, interest areas, salary floor, locations. Overrides resume-inferred defaults during `/jobs find`. |

## User resume — the source of truth

This repo has **no separate profile file**. Everything Claude needs about the user comes from their resume in `resumes/`. Read whatever's there (PDF / .tex / .md / .docx) whenever a flow needs:

- The user's **name** (top of resume) → used by `/jobs` Step 0 to verify the right LinkedIn account is active in Chrome.
- **Target job titles** (resume headline + experience entries) → used by `/jobs find` for searches.
- **Top skills / keywords** (skills section + repeated terms across bullets) → used by `/jobs find` for scoring.
- A short **third-person pitch** (current title + employer + years experience + 2-3 specializations) → used by `/jobs outreach` when drafting the first DM.

If `resumes/` is empty (only the README), stop the flow and tell the user to drop their resume in there before running anything.

If multiple resumes are present, read all of them and use the union of titles + skills.

## Optional search profile — what the user wants

If `resumes/search_profile.md` exists, read it alongside the resume. It's free-form prose: must-haves, deal-breakers, interest areas, salary floor, locations, anything that's not on the resume. Apply it as overlays on top of the resume-inferred plan in `/jobs find`. **The search profile wins on conflict** — the resume describes what the user *can* do, the search profile describes what they *want* to do. See `_shared.md` and `find.md` for how it's applied.

**Build the search profile WITH the user, not as homework.** When introducing the workspace, when starting a flow that would benefit from one, or any time the user mentions a preference ("I only want remote roles", "no crypto", "I'm into climate tech"), offer to build the file together. Don't tell them to "drop a search_profile.md" as if they have to write it themselves. Say something like:

> "Want me to build a search profile with you? Tell me your must-haves (locations, salary floor, level, role types), deal-breakers (industries, role shapes), and interest areas — I'll write `resumes/search_profile.md` for you."

When the user describes preferences in chat, capture them and **write `resumes/search_profile.md` using the Write tool** in the structured format from `resumes/README.md` (Must-haves / Interests / Deal-breakers / Salary). If the file already exists, read it first, then merge the new preferences in.

If the user clearly doesn't want a profile and wants to skip, fine — fall back to resume-only inference and don't keep nagging.

## Hard rules

1. **`job_tracker.csv` is the single source of truth.** When updating, preserve every untouched row exactly.
2. **URL column is mandatory.** Every new job row must have a working apply link. Ask or search before saving.
3. **Application folders are dated** if created: `applications/<YYYY-MM-DD>_<Company>_<Role>/`. Sanitize names (spaces → underscores, strip commas). This repo doesn't ship application infra by default; create the folder only if the user explicitly asks.
4. **Date format:** always `YYYY-MM-DD`.
5. **Render tracker contents as a clean markdown table** — never raw CSV.
6. **NEVER use em-dashes (—) in any user-facing message the user sends** — emails, LinkedIn DMs, follow-ups, subject lines, anything outgoing. Use commas, periods, parentheses, or split sentences instead. Em-dashes are a tell of AI-written text. Scan every draft for `—` before showing it; rewrite if found. Internal notes, tracker, contacts files are fine.
7. **For outreach DMs: you draft the message, the user sends it.** Browser MCP can navigate to the thread and type the message body, but you do NOT click Send without explicit confirmation. Standing flow: (a) navigate to the thread, (b) type the message body, (c) STOP and ask: "Ready to send?", (d) wait for the user to confirm before clicking Send.
8. **Connection requests: always "Send without a note", never personalized.** LinkedIn rate-limits personalized invites; bulk connection requests must always go through the "Send without a note" button. **One confirmation per company batch** — show the user the full list of names, get a single "y" to send the whole batch. Per-company quota = `max(0, (10 − count_1st_degree) × 5)`. **No global weekly cap** — keep going until LinkedIn pushes back (CAPTCHA, rate-limit notice). See `.claude/commands/jobs/outreach.md` Step 2C and Step 4B.
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

Match the user's voice. Skip over-explanation. Default to terse, technical responses. When drafting outreach DMs, follow the templates in `outreach.md` and build the pitch from the user's resume in `resumes/` — do not invent claims about their experience.
