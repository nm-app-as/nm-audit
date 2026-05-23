შენ ხარ Senior Full-Stack Engineer. ამ ფაზაში ფიქსავ 🟢 Medium და Low სიმძიმის საკითხებს —
code quality, DRY violations, complexity, documentation gaps, refactoring.

⚠️ Medium fix-ისთვის approval სჭირდება. სანამ ფიქსავ — ჩამოწერე რას ცვლი, რა შეიძლება გატყდეს.

---

## ნაბიჯი 0 — კონტექსტი

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"
TEST_CMD=$(jq -r '.commands.test_coverage // .commands.test // "npm test"' audit/.config.json)
LINT_CMD=$(jq -r '.commands.lint // "npm run lint"' audit/.config.json)

cat "$AUDIT_DIR/00_PROGRESS_TRACKER.md"
cat "$AUDIT_DIR/00_REGRESSION_RISKS.md"

# Checkpoint
git add .
git commit --allow-empty -m "$(cat <<'EOF'
checkpoint: before medium/refactoring fixes

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## ნაბიჯი 1 — Medium Issues-ის სია

წაიკითხე `00_PROGRESS_TRACKER.md`. გამოყავი 🟢 Medium და Low.
დაახარისხე: Code quality > Documentation > DX improvements.

---

## ნაბიჯი 2 — Approval Protocol

თითოეული medium fix-ისთვის სანამ იწყებ:

```
🔧 ვაპირებ: [issue სახელი]
📁 ფაილები: [affected files]
🔄 ცვლილება: [რა შეიცვლება]
⚠️ რისკი: [LOW / MEDIUM]
🛡 დაცვა: [რომელი ტესტები დაიჭერს regression-ს]

გავაგრძელო?
```

გააგრძელე მხოლოდ user-ის "yes" / "კი" / "გავაგრძელო"-ს შემდეგ.

---

## ნაბიჯი 3 — თითოეულ Medium Issue-ზე

### 3.1 — სწორი specialized agent
- Refactoring → `architect-reviewer`
- Code quality → `code-reviewer` ან `code-simplifier`
- Documentation → `writer` ან `document-specialist`
- Test coverage gap → `test-generator`

### 3.2 — TDD
```
1. ტესტი ❌  (თუ კოდი იცვლება)
2. ფიქსი
3. ✅
4. სხვა ტესტებიც ✅
```

### 3.3 — Timeout წესი
30 წუთში ვერ წყვეტ → `❌ გადაიდო` + მიზეზი.

### 3.4 — დასრულება

ჯერ ტესტი + lint — თუ ერთერთი ვარდება, commit ბლოკდება:

```bash
$TEST_CMD && $LINT_CMD || { echo "❌ tests/lint failed — commit ბლოკდება. გაასწორე ჯერ."; exit 1; }
```

tracker-ის განახლება (ხელით edit):
- `$AUDIT_DIR/00_PROGRESS_TRACKER.md`
- `$AUDIT_DIR/00_DECISIONS_LOG.md` ← refactoring decisions ADR-ად

commit Co-Authored-By trailer-ით (`refactor:` ან `docs:`):

```bash
git add .
git commit -m "$(cat <<'EOF'
refactor: [issue title]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## ნაბიჯი 4 — Code Quality Metrics (სავალდებულო)

ჩაუშვი ბოლოს:
```bash
# TypeScript any
grep -rn ": any\|as any" src --include="*.ts" | grep -v ".test.ts" | wc -l

# Files > 400 ხაზი
find src -name "*.ts" ! -name "*.test.ts" | xargs wc -l | sort -rn | head -10

# ESLint clean
$LINT_CMD
```

ჩაწერე შედეგები `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`-ში.

---

## სესიის ბოლოს ანგარიში

```markdown
## ✅ შესრულდა (Medium/Low)
- [ფაილი + ხაზი]: ცვლილება + commit hash

## ❌ გადაიდო
- [ფაილი]: მიზეზი + next sprint

## 📊 Code Quality Delta
| მეტრიკა | იყო | გახდა |
|---------|-----|-------|
| TypeScript `any` count | X | X |
| Files > 400 lines | X | X |
| ESLint errors | X | X |

## ⚠️ ყურადღება
- [risks, ADR-ები]
```

---

## შემდეგი ნაბიჯი

```
/audit:3b_verify_medium
```
