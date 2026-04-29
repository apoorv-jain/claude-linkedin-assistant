# Requirements & Setup

What you need to use this repo.

## 1. Git (mandatory)

Pre-installed on macOS / most Linux distros. Verify:

```bash
git --version
```

## 2. Claude Code (mandatory)

The CLI that runs the `/jobs` flows. Install: <https://docs.claude.com/en/docs/claude-code>

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
claude --version && echo "claude ✓"
```

Then open the repo in Claude Code:

```bash
cd claude-linkedin-assistant
claude
```

In Claude Code, run `/jobs` to start.

## First-run setup

1. Copy the profile templates:
   ```bash
   cp profile/profile.template.md profile/profile.md
   cp profile/personal_info.template.json profile/personal_info.json   # optional
   ```
2. Fill in `profile/profile.md` with your name, target roles, top skills, and a short pitch.
3. Make sure your LinkedIn account is signed in inside the Chrome extension's browser.
4. Run `/jobs` in Claude Code. Step 0 verifies Chrome + LinkedIn before anything else.
