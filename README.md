# Claude Code Audit Pipeline

A 7-phase code audit workflow for Claude Code, packaged as namespaced slash commands (`/audit:*`). Runs security, architecture, testing, performance, and CI/CD reviews — then writes structured reports, fixes findings by severity, and produces an investment-grade verdict with an HTML dashboard.

> Languages: this README is English-first. Georgian section is at the bottom (ქართული ვერსია ქვემოთაა).

---

## Quick Install (AI-assisted, recommended)

Open **Claude Code** inside any project and paste this to your agent:

```
Install the audit pipeline from https://github.com/nm-app-as/nm-audit
Ask me first whether to install globally or locally, then follow INSTALL.md exactly.
```

The agent will:
1. Read [INSTALL.md](INSTALL.md)
2. **Ask you: global or local install?**
   - **Global** → `~/.claude/commands/audit/` (available in every project on this machine)
   - **Local** → `./.claude/commands/audit/` (only this project, can be committed to your repo)
3. Download the `audit/` folder into the right place
4. Verify the install and tell you how to start

After install, run `/audit:0_how_to_use` for the full pipeline guide, or `/audit:init` to detect your stack.

---

## Manual Install

If you prefer to do it yourself:

### Global (machine-wide)

```bash
# macOS / Linux
mkdir -p ~/.claude/commands
git clone https://github.com/nm-app-as/nm-audit.git /tmp/claude-audit-pipeline
cp -R /tmp/claude-audit-pipeline/audit ~/.claude/commands/audit
rm -rf /tmp/claude-audit-pipeline
```

```powershell
# Windows PowerShell
$tmp = "$env:TEMP\claude-audit-pipeline"
git clone https://github.com/nm-app-as/nm-audit.git $tmp
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\commands" | Out-Null
Copy-Item -Recurse -Force "$tmp\audit" "$env:USERPROFILE\.claude\commands\audit"
Remove-Item -Recurse -Force $tmp
```

### Local (current project only)

```bash
# macOS / Linux
mkdir -p .claude/commands
git clone https://github.com/nm-app-as/nm-audit.git /tmp/claude-audit-pipeline
cp -R /tmp/claude-audit-pipeline/audit .claude/commands/audit
rm -rf /tmp/claude-audit-pipeline
```

```powershell
# Windows PowerShell
$tmp = "$env:TEMP\claude-audit-pipeline"
git clone https://github.com/nm-app-as/nm-audit.git $tmp
New-Item -ItemType Directory -Force -Path ".\.claude\commands" | Out-Null
Copy-Item -Recurse -Force "$tmp\audit" ".\.claude\commands\audit"
Remove-Item -Recurse -Force $tmp
```

---

## What's Inside

| File | Purpose |
|------|---------|
| `audit/init.md` | One-time stack detection → writes `audit/.config.json` |
| `audit/0_how_to_use.md` | Full pipeline guide (read this first) |
| `audit/1_audit.md` | Full audit scan, creates `audit/YYYY-MM-DD/` |
| `audit/2_after_audit.md` | Restructure findings into per-category files |
| `audit/3_fix_critical.md` | Fix 🔴 Critical issues (CVSS 9+) |
| `audit/3b_verify_critical.md` | QA verification of critical fixes |
| `audit/3_fix_high.md` | Fix 🟡 High issues (CVSS 7–8) |
| `audit/3b_verify_high.md` | QA verification of high fixes |
| `audit/3_fix_medium.md` | Fix 🟢 Medium / refactoring (with approval) |
| `audit/3b_verify_medium.md` | QA verification of medium fixes |
| `audit/4_run_project.md` | Runtime checks (endpoints, load test, deeper scan) |
| `audit/5_attack.md` | Red team / blue team adversarial audit |
| `audit/6_architecture_evolution.md` | Architecture evolution plan |
| `audit/7_final.md` | Investment verdict + HTML dashboard |
| `audit/quick_check.md` | One-category quick audit (no full pipeline) |
| `audit/abort.md` | Safe rollback to main, preserves audit branch + output |

---

## Usage

Inside Claude Code, after install:

```
/audit:init                  # First time only — detects stack
/audit:0_how_to_use          # Read the pipeline guide
/audit:1_audit               # Start a full audit
/audit:quick_check security  # Or run a quick single-category check
```

Each phase reads its predecessor's output. One audit cycle = one branch (`audit/YYYY-MM-DD`). `main` stays untouched until you merge.

---

## Updating

Re-run the install one-liner with the same global/local choice. The agent will overwrite the existing `audit/` folder with the latest version.

## Uninstall

- **Global:** `rm -rf ~/.claude/commands/audit` (or delete the folder on Windows)
- **Local:** `rm -rf .claude/commands/audit`

---

## License

MIT — see [LICENSE](LICENSE).

---

## ქართული ვერსია

ეს არის Claude Code-ის აუდიტის pipeline — 7 ფაზის სრული workflow რომელიც ამოწმებს უსაფრთხოებას, არქიტექტურას, ტესტებს, performance-ს და CI/CD-ს, შემდეგ წერს სტრუქტურირებულ ანგარიშებს, ასწორებს ნაპოვნ პრობლემებს severity-ის მიხედვით, და ბოლოს გამოიტანს investment-დონის verdict-ს HTML dashboard-ით.

### სწრაფი დაყენება (AI-აგენტით)

გახსენი **Claude Code** ნებისმიერ პროექტში და შეუბარე:

```
დააინსტალირე აუდიტის pipeline ლინკიდან: https://github.com/nm-app-as/nm-audit
ჯერ მკითხე გლობალურად დავაყენო თუ ლოკალურად, შემდეგ მიყევი INSTALL.md-ს ზუსტად.
```

აგენტი:
1. წაიკითხავს [INSTALL.md](INSTALL.md)-ს
2. **გკითხავს: გლობალურად თუ ლოკალურად დავაყენო?**
   - **გლობალური** → `~/.claude/commands/audit/` (ყველა პროექტში ხელმისაწვდომი)
   - **ლოკალური** → `./.claude/commands/audit/` (მხოლოდ ამ პროექტში)
3. გადმოწერს `audit/` ფოლდერს და დააყენებს სწორ ადგილას
4. შეამოწმებს და გეტყვის როგორ დაიწყო

დაყენების შემდეგ გაუშვი `/audit:0_how_to_use` გზამკვლევისთვის, ან `/audit:init` რომ stack იყოს გამოვლენილი.

### გამოყენება

```
/audit:init                  # ერთხელ — გამოავლენს stack-ს
/audit:0_how_to_use          # სრული გზამკვლევი
/audit:1_audit               # სრული აუდიტის დაწყება
/audit:quick_check security  # ერთი კატეგორიის სწრაფი შემოწმება
```

დეტალური ლოგიკა და ფოლდერის სტრუქტურა — `audit/0_how_to_use.md`-ში.
