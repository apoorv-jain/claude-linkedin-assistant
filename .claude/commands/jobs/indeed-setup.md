# INDEED-SETUP (enable Indeed / optimize profile)

`/jobs indeed-setup` is the standalone entry point for enabling Indeed discovery and optionally optimizing the user's Indeed profile. It can be run:

- After `/jobs setup` if the user chose "later" for profile optimization.
- Any time the user wants to enable or retry Indeed.
- To re-optimize the profile after updating a resume.

---

## Step 0 — Verify Indeed login

Navigate Chrome to `https://www.indeed.com/`. Read the page.

- **Signed in** (user's name/avatar visible, account menu present) → continue.
- **Not signed in** → "Not signed into Indeed. Please sign in at indeed.com in this browser, then tell me 'done'." Wait. Re-check.
- **CAPTCHA / blocked** → update `resumes/search_profile.md` by preserving the existing `Discovery` value if present, or writing `Discovery: disabled` if no Indeed state exists; set `Profile optimization: blocked`; tell the user "Indeed is blocking automation right now. Try again later with `/jobs indeed-setup`." Stop.

---

## Step 1 — Read source material

Read all resume files in `resumes/` (exclude `README.md` and `search_profile.md`).
Read `resumes/search_profile.md` if it exists.

Extract from the resume:

- **Name** (top of document)
- **Current title + employer**
- **Target roles** (headline + experience entries + adjacent titles)
- **Top skills / keywords** (skills section + repeated terms)
- **Education** (degrees, schools, years)
- **Summary** (if present)

From the search profile (if present):

- **Desired locations** and workplace types
- **Salary floor**
- **Job type preferences**
- **Deal-breakers / exclusions**

---

## Step 2 — Ask what the user wants

> I can do two things:
> 1. **Enable Indeed discovery only** — future `/jobs find` runs will include Indeed searches.
> 2. **Enable discovery AND optimize your Indeed profile** — I'll update your headline, summary, experience, skills, and preferences on Indeed from your resume. I'll show every change before saving.
>
> Which do you prefer? (1 / 2)

### Option 1 — Discovery only

- Write `Indeed: yes` and `Discovery: enabled` to `resumes/search_profile.md`.
- Record `Profile optimization: skipped`.
- Print: "Indeed discovery enabled. Your next `/jobs find` will include Indeed."
- Stop.

### Option 2 — Discovery + profile optimization

- Write `Indeed: yes` and `Discovery: enabled`.
- Do not write `Profile optimization: completed` yet. That status is only written after all confirmed profile saves finish.
- Continue to Step 3.

---

## Step 3 — Audit current profile vs resume

Navigate to `https://profile.indeed.com/`. The landing page shows contact info, a visibility banner, and section cards.

**`get_page_text` may not capture the resume editor (JS SPA) — use `screenshot` + `read_page` as fallback.**

Open the structured Indeed Resume at `https://profile.indeed.com/resume` and read it. It may be empty, stale, or partially filled.

Build a diff table and show the user:

```
Section | On Indeed now | Should be (from resume) | Action
```

Wait for the user to review before proceeding.

---

## Step 4 — Fill / update each section

Walk sections in order. For each section, **show the user what will be saved and get an explicit "yes" before clicking Save.**

### 4.1 Contact info
Name, email, phone, location — only fill what is visible in the resume or already on Indeed. Do not invent contact details.

### 4.2 Headline
Derive from current/target role and strongest resume keywords. Keep it factual and resume-backed.

### 4.3 Summary / About
Convert the resume's summary into concise first-person profile text. No em-dashes (scan before saving). Every claim must be resume-backed.

### 4.4 Work experience
Titles, companies, dates, and bullets from the resume. It is OK to shorten bullets for Indeed readability, but never change facts or inflate scope. Never fabricate experience.

### 4.5 Education
From the resume. Only add details the resume supports.

### 4.6 Skills
Derive tags from the resume's Skills section. Prefer concrete skills, tools, and methods named in the resume. Remove stale or off-target tags not supported by the resume.

### 4.7 Job preferences
Fill from `search_profile.md` when available:
- **Desired titles:** from the resume's target roles (3-6 titles max).
- **Job type:** from search profile or default to Full-time.
- **Minimum base pay:** from search profile salary floor if set. If no floor, skip.
- **Workplace type:** tick all types the user is open to (Remote, Hybrid, On-site) per search profile or resume location signals.
- **Commute:** set a reasonable commute range if the user has a local location.

### 4.8 Resume (the override trap)
How Indeed works: the structured "Indeed Resume" is the searchable/primary resume and **overrides any uploaded file**. The profile resume page (`/resume`) has no "upload to replace" — only Download / Share / Change language / Delete.

So when a stale or wrong Indeed Resume exists, the fix is to **rebuild its content in place** — update Summary, fix experience entries, update Skills. No upload needed.

**Deleting the Indeed Resume is destructive → user only, never Claude.**

Default: rebuild the structured Indeed Resume to mirror the user's resume. Record the outcome in `search_profile.md`.

### 4.9 Qualifications
Use only resume-backed facts:
- Most recent work experience from the resume.
- Education (highest degree first).
- Skills: 15-25 high-signal tags from the resume Skills section.

Do **not** fill:
- Licenses / certifications unless listed in the resume.
- Languages unless present in the resume or confirmed by the user.

### 4.10 Block employers / hide jobs
Default to no blocks and no hidden-job rules unless the user names specific exclusions. These controls can suppress good jobs.

### 4.11 Ready to Work
Set only if:
- The search profile says immediate availability, OR
- The user explicitly asks.

If the search profile says a notice period (e.g. "2 weeks notice") and the user hasn't explicitly asked, do not mark "available to start immediately."

### 4.12 Visibility
Read the current visibility state. Recommend Public / searchable while actively job-hunting. The user's privacy choice — confirm before changing.

---

## Step 5 — Career Scout note

Career Scout is Indeed's personalized job recommender and it is **mobile-app only (iOS/Android)** — it cannot be driven from Chrome. After the profile is updated, tell the user once:

> Your Indeed profile is optimized. The best matches now come from Career Scout, which is only in the Indeed phone app. Open the app, run Career Scout, and paste any roles it surfaces here. I'll verify them and add them to your tracker.

---

## Step 6 — CAPTCHA handling

Indeed runs Cloudflare + DataDome. If a CAPTCHA / "verify you are human" appears while saving:

- **STOP.** Tell the user: "Indeed wants a human check. Solve it, then tell me 'done' and I'll continue."
- Never try to solve it via automation.
- If the user cannot complete the check or the block persists, update `resumes/search_profile.md` with `Profile optimization: blocked` while preserving `Discovery: enabled`, then stop and explain that `/jobs indeed-setup` can retry later.

---

## Step 7 — Save state

Update `resumes/search_profile.md`:

```markdown
## Indeed
- Discovery: enabled
- Profile optimization: completed
```

Do not write resume-derived profile prose back to any committed file. The resume remains the source of truth.

---

## Failure rules summary

- **Not logged in:** ask the user to sign in and say `done`.
- **CAPTCHA / block:** stop Indeed setup, write `Profile optimization: blocked` to `resumes/search_profile.md` while preserving any existing discovery state, and explain `/jobs indeed-setup` can retry.
- **File upload / photo:** user handles it manually. Stop and give them the path.
- **Unclear field:** ask or skip, do not guess.
- **Every profile save requires explicit user confirmation.** Show the proposed content, wait for "yes."
