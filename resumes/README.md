# resumes/

Drop your resume here. The `/jobs` flows read whatever you put in this folder to figure out:

- Your **name** (used to verify the right LinkedIn account is active in Chrome before any flow runs).
- Your **target job titles + top skills** (used by `/jobs find` to search and score jobs).
- A short **third-person pitch** about your background (used by `/jobs outreach` when drafting the first DM).

## Supported formats

Any of these:

- `resume.pdf` (most common)
- `resume.tex` (LaTeX source — Claude reads it as plain text)
- `resume.md`
- `resume.docx`

You can drop multiple resumes here if you have role-specific versions (e.g. `resume_data_scientist.pdf`, `resume_ml_engineer.pdf`). The flows read all of them and use the union of titles + skills.

## What's NOT in this folder

- ❌ A separate "profile" or "personal info" file. Your resume already has your name, contact info, and experience. Don't duplicate it.
- ❌ Cover letters. This repo doesn't apply on your behalf.

## Privacy

The contents of this folder are `.gitignore`d by default. Your resume stays local. Only this README is committed.

If you fork this repo to a private repo and want to commit your resume too, edit `.gitignore`.
