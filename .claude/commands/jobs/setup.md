# SETUP (first-run configuration)

`/jobs setup` walks the user through platform configuration. It runs automatically the first time `/jobs find` or `/jobs daily` is invoked without platform config, or manually any time.

---

## Step 1 — Resume check

List `resumes/`. Filter out `README.md` and `search_profile.md`.

- **Zero resume files** → follow the same warm prompt from `jobs.md` Step 0A. Wait for the user to provide one before continuing.
- **Resume found** → extract name, current title, and at least one skill. Hold the name for Step 2.

---

## Step 2 — Chrome + LinkedIn verification

Run the same Chrome + LinkedIn check from `jobs.md` Step 0B:

1. Verify Chrome extension is connected.
2. Navigate to `https://www.linkedin.com/in/me/`.
3. Confirm the signed-in profile name matches the resume name.
4. Print: `✓ LinkedIn: <name>`

If any check fails, stop with the same error messages as `jobs.md` Step 0.

For the Chrome-extension failure case, use the full recovery checklist from `jobs.md` Step 0B so first-time users know how to install/enable the extension and retry setup.

---

## Step 3 — Indeed opt-in

Ask:

> Do you want to include Indeed in your job searches? It adds Indeed as another source alongside LinkedIn and WebSearch. Requires being signed into indeed.com in this Chrome browser. (yes / no / later)

### User says no or later

- Read `resumes/search_profile.md` if it exists (to preserve existing content).
- Add or update the `## Platforms` section:
  ```
  ## Platforms
  - LinkedIn: yes
  - Indeed: no
  ```
- Add or update the `## Indeed` section:
  ```
  ## Indeed
  - Discovery: disabled
  ```
- Print: `Indeed skipped. You can enable it later with /jobs indeed-setup.`
- Skip to Step 5.

### User says yes

1. Navigate to `https://www.indeed.com/` in Chrome.
2. Read the page.
3. **Signed in** (user's name/avatar visible, account menu present):
   - Print: `✓ Indeed: signed in`
   - Write `Indeed: yes` and `Discovery: enabled` to search profile (see state format below).
   - Continue to Step 4.
4. **Not signed in** (shows "Sign in" / login page):
   - Tell the user: "Not signed into Indeed. Please sign in at indeed.com in this browser, then tell me 'done'."
   - Wait for confirmation.
   - Re-navigate and re-check.
   - If now signed in → write state, continue to Step 4.
   - If still not signed in → "Still not seeing a signed-in state. Try again, or say 'skip' to disable Indeed for now."
5. **CAPTCHA / blocked / blank page:**
   - Print: "Indeed is blocking automation right now. I'll mark it as skipped. You can enable it later with `/jobs indeed-setup`."
   - Write `Discovery: disabled` and `Profile optimization: blocked`.
   - Skip to Step 5.

---

## Step 4 — Indeed profile optimization (optional)

Only reached if Indeed discovery was successfully enabled in Step 3.

Ask:

> Do you want me to help optimize your Indeed profile from your resume? This can improve Indeed's job matching, but it edits public-facing profile fields like headline, summary, skills, qualifications, preferences, and Ready to Work. I will show each change before saving. (yes / no / later)

### User says yes

- Dispatch to `/jobs indeed-setup` (read and execute `.claude/commands/jobs/indeed-setup.md`).
- After it completes, read `resumes/search_profile.md` and preserve the actual `Profile optimization` value written by `indeed-setup`.
- Only record `Profile optimization: completed` if the profile optimization branch actually finished all confirmed profile saves. Do not overwrite `skipped`, `later`, or `blocked`.

### User says no

- Record `Profile optimization: skipped`.

### User says later

- Record `Profile optimization: later`.
- Print: "You can run `/jobs indeed-setup` any time to optimize your Indeed profile."

---

## Step 5 — Summary

Print:

```
Setup complete:
  LinkedIn: ✓ <name>
  Indeed discovery: enabled / disabled
  Indeed profile: completed / skipped / later / blocked

Run /jobs daily to start, or /jobs find for discovery only.
```

---

## Writing state to search_profile.md

Read `resumes/search_profile.md` first if it exists — preserve all existing content (Must-haves, Interests, Deal-breakers, Salary, etc.).

Add or update these two sections:

```markdown
## Platforms
- LinkedIn: yes
- Indeed: yes

## Indeed
- Discovery: enabled
- Profile optimization: skipped
```

If the file does not exist, create it with just the Platforms and Indeed sections. The user can fill in the rest later or via `/jobs find`'s search-profile builder.

Never overwrite user-written preferences when updating platform config.
