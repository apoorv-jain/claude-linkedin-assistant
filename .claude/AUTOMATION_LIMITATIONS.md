# Automation Limitations

Honest map of what `/jobs` automates vs. what you have to do yourself. Read this before letting anything run end to end.

**TL;DR: this repo only automates the safe, repetitive parts.** First DMs to existing connections, tracker reads/writes, job discovery. Everything else is on you.

---

## ✅ Claude IS reliable at

- **Reading and querying the tracker.** `/jobs check`, filtered queries, dashboards.
- **Sending first cold DMs to your existing 1st-degree LinkedIn connections** (short templated intros).
- **Job discovery + scoring** via LinkedIn search and web search, when the profile is well defined.
- **Adding, updating, and writing rows in `job_tracker.csv`** when explicitly told what changed.
- **Drafting message bodies.** Composing the text itself.

## ⚠️ Out of scope by design (this repo will NOT do these)

### 1. Sending connection requests

This repo deliberately does not send `Connect` requests. If outreach surfaces only 2nd-degree candidates, they're logged as `Connection Pending` and you send the requests manually. Connection requests have a personal quality control layer LinkedIn rate-limits aggressively, and bulk-sending can damage your reputation.

### 2. Replying to DMs

When someone replies to a first DM you sent, the conversation is yours to handle. Reply tone, claims about your background, decisions about whether to share a resume — all judgment calls. This repo doesn't draft replies.

### 3. Follow-up nudges

Standard practice is a 3-5 day follow-up if outreach goes stale. This repo doesn't send those. If you want them, send manually.

### 4. Application submission, resume tailoring, cover letters

Out of scope. Tailoring a resume to a JD requires reading the job description carefully, deciding which bullets to swap, and verifying nothing was fabricated. Submitting an application requires sensitive judgment calls (salary, sponsorship, custom screening, demographic disclosures). Do these yourself.

### 5. File uploads

LinkedIn DM attach, Gmail compose, Greenhouse, Ashby, and most ATS forms block automated file upload via both browser MCP and computer-use. The macOS file picker sheet inherits Chrome's tier-`read` restriction. Bottom line: there's no working automated path to attach a file. This repo's flows are designed not to need uploads.

---

## ⚠️ Claude is RISKY at, supervise even within scope

### 1. Picking the right LinkedIn job link

Job titles often look identical across teams; the URL can point to a different role than expected. Always verify the URL before sharing it with a contact. Real failure mode: an outreach message sent with the wrong job link.

### 2. Multi-window / multi-tab Chrome state

The browser MCP is connected to one Chrome window. If your Chrome is in the background and another Chrome window is in front, automation may target the wrong window. The right Chrome window must be brought forward manually.

### 3. LinkedIn rate limiting / CAPTCHAs

If you run a lot of searches or send many DMs in a short window, LinkedIn may throttle or show a CAPTCHA. The find and outreach flows stop and tell you when this happens. Slow down or run in smaller batches.

### 4. Outreach message content

The DM templates use the third-person pitch you wrote in `profile/profile.md`. If your pitch is sloppy, the DM is sloppy. Read every drafted DM before approving Send. Watch for em-dashes (a tell of AI-written text) — the flows scan for them but read anyway.

---

## 🚦 Supervision rule of thumb

| Action | Supervision |
|---|---|
| `/jobs check` (read tracker) | Low |
| `/jobs find` (discover new jobs) | Low |
| `/jobs add` (add a job) | Low |
| `/jobs update` (status change) | Low |
| First cold DM to a 1st-degree connection | **Medium** — read the draft, confirm Send |
| Anything beyond first DM | **Out of scope** — do it yourself |

---

## General pattern

When automation hits a wall on a required field, upload, or judgment call, the flow will:

1. Explain what's blocked.
2. Tell you what to do manually.
3. Resume from there once you confirm.

Standing rules in `CLAUDE.md` and `.claude/commands/jobs/_shared.md` exist because of these limits (em-dashes, no connection requests, no file uploads, explicit-Send confirmation). When a new failure mode shows up, add it here.
