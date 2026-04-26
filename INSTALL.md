# Install Instructions (for AI Agents)

This file is the canonical install recipe. If you are an AI agent (Claude Code or similar) and the user asked you to install this repository, follow these steps **in order, without skipping**.

---

## Step 1 — Ask the user: global or local?

**Do not skip this step.** Before downloading anything, ask the user this exact question (translate to their language if needed, but keep both options):

> Should I install this **globally** (available in every project on this machine, at `~/.claude/commands/audit/`) or **locally** (only this project, at `./.claude/commands/audit/`, can be committed to your repo)?

Wait for the user's answer. Accept synonyms:
- Global: `global`, `globally`, `machine-wide`, `everywhere`, `გლობალური`, `გლობალურად`
- Local: `local`, `locally`, `project`, `here`, `ლოკალური`, `ლოკალურად`, `პროექტში`

If the answer is ambiguous, ask once more before proceeding.

---

## Step 2 — Resolve the install target

Based on the user's choice, set the target directory:

| Choice | Target directory |
|--------|------------------|
| Global (macOS / Linux) | `~/.claude/commands/audit` |
| Global (Windows)       | `%USERPROFILE%\.claude\commands\audit` (PowerShell) or `$HOME/.claude/commands/audit` (Git Bash) |
| Local (any OS)         | `./.claude/commands/audit` (relative to current working directory) |

**If the target directory already exists**, ask the user:

> An existing `audit/` install already exists at `<TARGET>`. Overwrite it with the latest version, or abort?

Only continue on explicit confirmation.

---

## Step 3 — Detect the source

The user gave you a GitHub URL. Use that URL for the clone. If the user gave only the repo name without an owner, ask for the full URL — do not guess.

Default branch: `main` (fall back to `master` only if `main` does not exist).

---

## Step 4 — Download and copy

Use a temporary clone, copy the `audit/` subfolder, then clean up. Pick the command set that matches the user's shell:

### macOS / Linux / Git Bash

```bash
TMP=$(mktemp -d)
git clone --depth 1 <REPO_URL> "$TMP/repo"
mkdir -p "$(dirname "<TARGET>")"
rm -rf "<TARGET>"
cp -R "$TMP/repo/audit" "<TARGET>"
rm -rf "$TMP"
```

### Windows PowerShell

```powershell
$tmp = New-Item -ItemType Directory -Path "$env:TEMP\claude-audit-$(Get-Random)"
git clone --depth 1 <REPO_URL> "$tmp\repo"
$target = "<TARGET>"
$parent = Split-Path -Parent $target
if (-not (Test-Path $parent)) { New-Item -ItemType Directory -Force -Path $parent | Out-Null }
if (Test-Path $target) { Remove-Item -Recurse -Force $target }
Copy-Item -Recurse -Force "$tmp\repo\audit" $target
Remove-Item -Recurse -Force $tmp
```

Substitute `<REPO_URL>` and `<TARGET>` with the values from Steps 2–3.

---

## Step 5 — Verify

Confirm the install worked:

```bash
ls "<TARGET>"          # macOS / Linux / Git Bash
```
```powershell
Get-ChildItem "<TARGET>"   # Windows PowerShell
```

You should see at least these files:
- `0_how_to_use.md`
- `1_audit.md`
- `init.md`
- `quick_check.md`
- and the rest of the `*.md` phase files

If the listing is empty or missing files, the install failed — surface the error to the user, do **not** silently move on.

---

## Step 6 — Report success

Tell the user, in their language:

> ✅ Audit pipeline installed at `<TARGET>`.
>
> Next steps:
> 1. Restart Claude Code (or run `/help` to refresh slash commands) so it picks up the new `/audit:*` commands.
> 2. Run `/audit:init` once to detect your project's stack.
> 3. Run `/audit:0_how_to_use` to read the full pipeline guide.

---

## Constraints (do not violate)

- **Never** install without asking global vs. local first.
- **Never** overwrite an existing `audit/` folder without explicit confirmation.
- **Never** install anywhere other than `~/.claude/commands/audit/` or `./.claude/commands/audit/`. Other paths break Claude Code's slash-command discovery.
- **Never** modify files inside `audit/*.md` during install — copy them as-is.
- **Never** run any other phase (`/audit:1_audit`, etc.) automatically after install. The user starts the pipeline themselves.
- If `git` is not available, fall back to downloading the repo tarball via the GitHub API (`https://github.com/<owner>/<repo>/archive/refs/heads/main.tar.gz`), extract, and copy the `audit/` folder. Do not invent other fallbacks.

---

## Quick Reference (single-line summary for the agent)

```
ASK global-or-local → RESOLVE target → CONFIRM if exists → CLONE shallow → COPY audit/ → VERIFY listing → REPORT success
```
