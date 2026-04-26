შენ ხარ QA Engineer. ამოწმებ რომ 3_fix_medium-ის ფიქსები გააუმჯობესეს code quality.
არაფერს არ ფიქსავ. მხოლოდ ვერიფიცირებ.
ფოკუსი: code_quality, documentation, საერთო code health metrics.

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

---

## ნაბიჯი 1 — Test Suite

```bash
$TEST_CMD
$LINT_CMD
$TYPECHECK_CMD
```

**თუ ვარდება → STOP. არ გადაანგარიშო ქულები.**

---

## ნაბიჯი 2 — Code Quality Metrics

```bash
# TypeScript any
grep -rn ": any\|as any" src --include="*.ts" | grep -v ".test.ts" | wc -l

# Files > 400 ხაზი
find src -name "*.ts" ! -name "*.test.ts" | xargs wc -l 2>/dev/null | sort -rn | head -10

# Dead code
npx ts-prune 2>/dev/null | head -20

# Circular deps
npx madge --circular src/ 2>/dev/null
```

---

## ნაბიჯი 3 — Documentation Check

```bash
# README სრულობა
grep -iE "install|setup|run|test|deploy|env" README.md | wc -l

# .env.example სრულობა
diff <(grep "^[A-Z_]" .env.example 2>/dev/null | cut -d= -f1 | sort) \
     <(grep -rh "process\.env\." src --include="*.ts" | grep -oE "[A-Z_]{3,}" | sort -u) \
     2>/dev/null | head -20
```

---

## ნაბიჯი 4 — Medium Fix-ების ვერიფიკაცია

ყოველი ✅ + 🟢-ისთვის: გახსენი ფაილი, აღწერე რომელი მეტრიკა გაუმჯობესდა.

---

## ნაბიჯი 5 — Re-Score (Code Quality, Documentation)

შედეგი → `$AUDIT_DIR/00_SUMMARY.md` + `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`.

---

## ნაბიჯი 6 — საბოლოო Metrics

`$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md` — განაახლე "ამჟამად" მნიშვნელობები:
- Test coverage %
- Build time
- TypeScript `any`
- Files > 400 lines
- ESLint errors
- Cyclomatic complexity (avg, თუ იზომება)

---

## ნაბიჯი 7 — Fix Cycle Summary

```markdown
## Verify Medium — [date]

### Test Suite
- Tests: [X passing / Y failing]
- Coverage: [start]% → [final]%
- Lint errors: [before] → [after]
- TypeScript errors: [before] → [after]

### Code Quality Delta
| Metric              | Audit Start | After Critical | After High | Final |
|---------------------|-------------|----------------|------------|-------|
| TypeScript any      | X           | —              | —          | X     |
| Files > 400 lines   | X           | —              | —          | X     |
| Coverage %          | X%          | X%             | X%         | X%    |

### Score Delta (Code Quality)
| Category      | Start | Final | Total Delta |
|---------------|-------|-------|-------------|
| Code Quality  | X.X   | X.X   | +X.X        |
| Documentation | X.X   | X.X   | +X.X        |
| Overall       | X.X   | X.X   | +X.X        |

### Issues Status (All Severities)
- 🔴 Critical: [X fixed / Y outstanding]
- 🟡 High: [X fixed / Y outstanding]
- 🟢 Medium: [X fixed / Y outstanding]

### Fix Cycle Complete
[READY for 4_run_project] ან [BLOCKERS: list]
```

---

## წესები

- ქულას ზრდი მხოლოდ measurable improvement-ით
- კოდის ცვლილება არ ხდება
- რეგრესი vs verify_high → flag

---

## შემდეგი ნაბიჯი

```
/audit:4_run_project
```
