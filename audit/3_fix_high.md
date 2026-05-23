შენ ხარ Senior Full-Stack Engineer. ამ ფაზაში ფიქსავ მხოლოდ 🟡 High სიმძიმის საკითხებს.
Critical უკვე გასწორდა 3_fix_critical-ში. Medium/Low — მომდევნო ფაზაში.

---

## ნაბიჯი 0 — კონტექსტი

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"
TEST_CMD=$(jq -r '.commands.test_coverage // .commands.test // "npm test"' audit/.config.json)
LINT_CMD=$(jq -r '.commands.lint // "npm run lint"' audit/.config.json)

cat "$AUDIT_DIR/00_PROGRESS_TRACKER.md"

# Checkpoint
git add .
git commit --allow-empty -m "$(cat <<'EOF'
checkpoint: before fixing high severity issues

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## ნაბიჯი 1 — High Issues-ის სია

წაიკითხე `00_PROGRESS_TRACKER.md`. გამოყავი მხოლოდ 🟡.
დაახარისხე: Testing gaps > Architecture > Performance > CI/CD.

თუ high არ არის — გადადი `/audit:3b_verify_high`-ზე.

---

## ნაბიჯი 2 — თითოეულ High Issue-ზე

### 2.1 — სანამ ფიქსავ
```bash
git diff --name-only HEAD
```
**High severity-სთვის: კომიტამდე ფაილების სია ჩამოწერე** — რომ user-მა დაინახოს რა შეცვლილა.

### 2.2 — სწორი specialized agent
- Testing gap → `test-generator` ან `test-engineer`
- Architecture → `architect-reviewer`
- Code quality → `code-reviewer`
- Performance → `general-purpose` + targeted analysis
- Frontend → `frontend-developer`

### 2.3 — TDD
```
1. failing test ❌
2. ფიქსი
3. ტესტი ✅
4. სხვა ტესტებიც ✅
```

### 2.4 — Timeout წესი
30 წუთში ვერ წყვეტ → tracker-ში `❌ დაბლოკილია` + მიზეზი → შემდეგზე გადადი.

### 2.5 — დასრულება

High severity: commit-ამდე ფაილების სია + test/lint blocking:

```bash
echo "Files changed: $(git diff --name-only HEAD | tr '\n' ', ')"

$TEST_CMD && $LINT_CMD || { echo "❌ tests/lint failed — commit ბლოკდება. გაასწორე ჯერ."; exit 1; }
```

tracker-ის განახლება (ხელით edit):
- `$AUDIT_DIR/00_PROGRESS_TRACKER.md`
- `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`
- `$AUDIT_DIR/00_DECISIONS_LOG.md`

commit Co-Authored-By trailer-ით:

```bash
git add .
git commit -m "$(cat <<'EOF'
fix: high — [issue title]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## ნაბიჯი 3 — სტატუსების flow

```
⬜ მოლოდინში   → 🔄 მიმდინარე
🔄 მიმდინარე   → ✅ შესრულდა    (ტესტი + commit)
🔄 მიმდინარე   → ❌ დაბლოკილია  (30 წთ+ ან გარე deps)
```

**წესები:**
- ნუ ფიქსავ Critical (უკვე გასწორებული) ან Medium/Low (მომდევნო ფაზა)
- High severity ცვლილება — ყოველთვის ჩამოწერე ფაილები commit-ამდე

---

## სესიის ბოლოს ანგარიში

```markdown
## ✅ შესრულდა (High)
- [ფაილი + ხაზი]: ცვლილება + commit hash

## ❌ დაბლოკილია
- [ფაილი]: მიზეზი + რა სჭირდება

## ⚠️ ყურადღება
- [რისკები, regression შესაძლებლობები]

## 🤖 გამოყენებული agents
- [agent სახელი]: [რისთვის]

## 📁 ფაილები (commit-მდე)
- [ყველა ცვლილებული ფაილი]
```

---

## შემდეგი ნაბიჯი

```
/audit:3b_verify_high
```
