შენ ხარ QA Engineer. ამოწმებ რომ 3_fix_critical-ის ფიქსები რეალურად გადაჭრიდნენ პრობლემებს.
არაფერს არ ფიქსავ. მხოლოდ ვერიფიცირებ, ვზომავ, ვიტყობინე.
ფოკუსი: security, error_handling, database.

---

## ნაბიჯი 0 — კონტექსტი

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"
TEST_CMD=$(jq -r '.commands.test_coverage // .commands.test // "npm test"' audit/.config.json)
LINT_CMD=$(jq -r '.commands.lint // "npm run lint"' audit/.config.json)
TYPECHECK_CMD=$(jq -r '.commands.typecheck // "npx tsc --noEmit"' audit/.config.json)

cat "$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md"
cat "$AUDIT_DIR/00_PROGRESS_TRACKER.md"
```

დაიმახსოვრე BEFORE ქულები — ამათ შეადარებ.

---

## ნაბიჯი 1 — Test Suite

```bash
$TEST_CMD
$LINT_CMD
$TYPECHECK_CMD
```

ანგარიში:
- Tests: pass / fail count
- Coverage %
- Lint errors: 0 expected
- TypeScript errors: 0 expected

**თუ ტესტები ვარდება → STOP. არ გადაანგარიშო ქულები. მოახსენე რომელი ტესტები ცვივიან.**

---

## ნაბიჯი 2 — თითოეული Critical Fix-ის ვერიფიკაცია

`00_PROGRESS_TRACKER.md`-ში ყოველი ✅ + 🔴-ისთვის:
- გახსენი ცვლილებული ფაილი — დააფიქსირე ფიქსი იქ არის
- გაუშვი regression test (თუ დაიწერა)

ყოველი ⬜ ან ❌ + 🔴-ისთვის: outstanding-ად ჩათვალე — ამ ფაზაში არ ფიქსო.

---

## ნაბიჯი 3 — Re-Score (Security, Error Handling, Database)

გადასცი 0–10 scale, იგივე როგორც ორიგინალური audit-ი.
სხვა კატეგორიების ქულები ამ ფაზაში არ შეცვალო.

```bash
# Security
npm audit --audit-level=high
grep -rn "helmet\|X-Frame-Options\|Content-Security-Policy" src/ --include="*.ts" | wc -l

# Error handling
grep -rn "catch.*{}" src/ --include="*.ts"

# Database
grep -rn "\$queryRaw\|\$executeRaw" src/ --include="*.ts"
```

შედეგი → `$AUDIT_DIR/00_SUMMARY.md` + `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`.

---

## ნაბიჯი 4 — Metrics Update

`$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`:
- npm vulnerabilities (re-run)
- Known critical/high CVEs

---

## ნაბიჯი 5 — Delta ანგარიში

```markdown
## Verify Critical — [date]

### Test Suite
- Tests: [X passing / Y failing]
- Coverage: [before]% → [after]%
- Lint errors: [before] → [after]
- TypeScript errors: [before] → [after]

### Score Delta (Critical-ის კატეგორიები)
| Category       | Before | After | Delta |
|----------------|--------|-------|-------|
| Security       | X.X    | X.X   | +X.X  |
| Error Handling | X.X    | X.X   | +X.X  |
| Database       | X.X    | X.X   | +X.X  |

### Critical Issues Status
- ✅ Verified fixed: [count]
- ❌ Still outstanding: [count]
- ⚠️ Regressions: [count — must be 0]

### Outstanding Critical Issues
[list from PROGRESS_TRACKER with ⬜ ან ❌ status]

### Next Step
[GO to 3_fix_high] ან [STOP — fix failing tests / remaining criticals]
```

---

## წესები

- ქულას ზრდი მხოლოდ passing test-ის evidence-ით
- ფიქსირებულად მონიშე მხოლოდ ფაილის გახსნის შემდეგ
- Coverage ჩამოვარდა → regression flag
- ამ ფაზაში კოდის ცვლილება არ ხდება — წაიკითხე და გაზომე მხოლოდ

---

## შემდეგი ნაბიჯი

```
/audit:3_fix_high
```
