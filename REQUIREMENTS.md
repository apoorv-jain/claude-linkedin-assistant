# Requirements & Setup

What you need to use this repo.

## 1. Git (mandatory)

Pre-installed on macOS / most Linux distros. Verify:

```bash
git --version
```

## 2. Claude Code desktop app (mandatory)

The desktop app is what runs the `/jobs` flows. Install: <https://docs.claude.com/en/docs/claude-code>

The web version at claude.ai/code will NOT work for this repo. The flows need to (a) read your resume files in `resumes/`, and (b) drive the Claude in Chrome extension — both require a local install.

**Open the repo:** launch the app, then File → Open Folder → pick the `claude-linkedin-assistant` folder.

## 3. Claude in Chrome extension (mandatory)

The `/jobs` flows drive LinkedIn through Chrome. Install the extension and sign into your LinkedIn account in that browser. <https://claude.com/claude-in-chrome>

## 4. CSV editor (recommended)

To view `job_tracker.csv` directly:

- **Numbers** (pre-installed on macOS)
- **LibreOffice** — `brew install --cask libreoffice` (macOS) or `apt install libreoffice` (Debian/Ubuntu)
- **VS Code + "Edit CSV" extension**

Or just use the `/jobs check` dashboard inside Claude Code.

## 5. GitHub CLI (optional)

Only if you want to fork and push your version of this repo:

```bash
brew install gh    # macOS
gh auth login
```

## Quick health check

```bash
git --version && echo "git ✓"
```

Then launch the Claude Code desktop app, File → Open Folder → pick `claude-linkedin-assistant`.

In the app, run `/jobs` to start.

## First-run setup

1. Put your resume into the `resumes/` folder. Drag it in from Finder, copy it from wherever you keep your latest version, whatever's easiest. Any format works: `.pdf`, `.tex`, `.md`, or `.docx`. Multiple files are fine if you have role-specific versions.

2. Make sure your LinkedIn account is signed in inside the Chrome extension's browser. The name on LinkedIn must match the name at the top of your resume — Step 0 of `/jobs` verifies this before anything else runs.

3. Run `/jobs` in Claude Code.
