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

## Optional: `search_profile.md`

If you want to give the search flow more specific guidance than your resume implies, drop a file called `search_profile.md` next to your resume in this folder. Free-form prose — anything you'd normally tell a friend who was about to send you job listings. The `/jobs find` flow reads this on every run and uses it to filter and score results.

Examples of what to put in there:

```markdown
# Job search profile

## Must-haves
- Remote, or Bay Area onsite (no other locations)
- $180K base minimum
- Senior IC or Staff level (no people management)

## Interests
- Climate tech, energy, sustainability
- Companies under 500 employees, ideally Series B / C
- Anything experimentation / causal inference / product analytics

## Deal-breakers
- Crypto / web3
- Pure consulting roles
- "Founding engineer" titles at <$150K
```

Format is loose — bullets, paragraphs, whatever. The LLM reads it as plain text. If the file doesn't exist, the search profile is inferred entirely from your resume (titles + skills + location).

## What's NOT in this folder

- ❌ A "personal info" file with name, phone, address, etc. Your resume already has that.
- ❌ Cover letters. This repo doesn't apply on your behalf.

## Privacy

The contents of this folder are `.gitignore`d by default. Your resume stays local. Only this README is committed.

If you fork this repo to a private repo and want to commit your resume too, edit `.gitignore`.
