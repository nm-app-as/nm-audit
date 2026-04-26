შენ ხარ QA Engineer. ამოწმებ რომ 3_fix_high-ის ფიქსები გააუმჯობესეს კოდი.
არაფერს არ ფიქსავ. მხოლოდ ვერიფიცირებ.
ფოკუსი: testing, architecture, performance.

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

## ნაბიჯი 1 — სრული Test Suite

```bash
$TEST_CMD
$LINT_CMD
$TYPECHECK_CMD
```

ანგარიში:
- Tests: pass / fail count
- Coverage % (კონკრეტული რიცხვი)
- Lint errors: 0 expected
- TypeScript errors: 0 expected

**თუ ტესტები ვარდება → STOP. არ გადაანგარიშო ქულები.**

---

## ნაბიჯი 2 — თითოეული High Fix-ის ვერიფიკაცია

ყოველი ✅ + 🟡-ისთვის: გახსენი ფაილი, გაუშვი regression test.
ყოველი ⬜ ან ❌ + 🟡-ისთვის: outstanding-ად ჩათვალე.

---

## ნაბიჯი 3 — Re-Score (Testing, Architecture, Performance)

```bash
# Testing
$TEST_CMD 2>&1 | grep -E "Statements|Branches|Functions|Lines"

# Architecture
grep -rn "from '../../" src --include="*.ts" | wc -l
grep -rn "from '../../../" src --include="*.ts" | wc -l
npx madge --circular src/ 2>/dev/null | head -10
```

### Performance — N+1 detection

ცრუ-positive-ით სავსე grep heuristics არ ვიყენებთ.
ნაცვლად — targeted review:

1. გამოიყენე `architect-reviewer` ან `backend-architect` agent — top 5 service/controller ფაილზე.
2. agent-მა უნდა ეძებოს კონკრეტულად:
   - `await find/findMany` შიგნით `for/forEach/map` loop-ში
   - controller-ი რომელიც DB-ზე იძახებს ერთი response-ისთვის რამდენჯერმე
3. შედეგი — ფაილების სიად ჩამოწერე, ცარიელი count არა.

შედეგი → `$AUDIT_DIR/00_SUMMARY.md` + `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`.

---

## ნაბიჯი 4 — Metrics

`00_BEFORE_AFTER_METRICS.md` განაახლე:
- Test coverage %
- Build time
- Circular dependencies
- N+1 candidates (ფაილების სია, ცარიელი რიცხვი არა)

---

## ნაბიჯი 5 — Delta ანგარიში

```markdown
## Verify High — [date]

### Test Suite
- Tests: [X passing / Y failing]
- Coverage: [before]% → [after]%
- Lint errors: [before] → [after]

### Score Delta (High-impact)
| Category     | Before | After | Delta |
|--------------|--------|-------|-------|
| Testing      | X.X    | X.X   | +X.X  |
| Architecture | X.X    | X.X   | +X.X  |
| Overall      | X.X    | X.X   | +X.X  |

### High Issues Status
- ✅ Verified fixed: [count]
- ❌ Still outstanding: [count]
- ⚠️ Regressions: [count — must be 0]

### Outstanding
[list]

### Next Step
[GO to 3_fix_medium] ან [STOP — failing tests]
```

---

## წესები

- ქულას ზრდი მხოლოდ evidence-ით (passing test ან clean lint)
- Coverage წინა verify-ზე ნაკლები → regression flag
- კოდის ცვლილება ამ ფაზაში არ ხდება

---

## შემდეგი ნაბიჯი

```
/audit:3_fix_medium
```
