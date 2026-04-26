შენ ხარ Senior Full-Stack Engineer. ამ ფაზაში ფიქსავ მხოლოდ 🔴 Critical სიმძიმის საკითხებს.
High, Medium, Low — გამოტოვე. ისინი მომდევნო ფაზებში ფიქსირდება.

---

## ნაბიჯი 0 — კონტექსტი

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"
TEST_CMD=$(jq -r '.commands.test_coverage // .commands.test // "npm test"' audit/.config.json)
LINT_CMD=$(jq -r '.commands.lint // "npm run lint"' audit/.config.json)

cat "$AUDIT_DIR/00_PROGRESS_TRACKER.md"
cat "$AUDIT_DIR/06_security/ISSUES.md" 2>/dev/null

# Checkpoint
git add .
git commit -m "checkpoint: before fixing critical issues" --allow-empty
```

---

## ნაბიჯი 1 — Critical Issues-ის სია

წაიკითხე `00_PROGRESS_TRACKER.md`. გამოყავი მხოლოდ 🔴.
დაახარისხე: Security > Data Integrity > Error Handling > Database.

თუ critical არ არის — გაჩერდი და მომახსენე. გადადი `/audit:3b_verify_critical`-ზე.

---

## ნაბიჯი 2 — თითოეულ Critical Issue-ზე

### 2.1 — დადასტურება სანამ შეეხები
```bash
git blame [affected_file] | tail -5
```
დარწმუნდი პრობლემა კვლავ არსებობს — შესაძლოა ვინმემ უკვე გამოასწორა.

### 2.2 — CVSS ტრიაჟი
- CVSS 9+ (Critical): ფიქსავ დაუყოვნებლივ
- CVSS 7-8 (High): ეს ფაზა არ ეკუთვნი — გამოტოვე

### 2.3 — სწორი specialized agent
- Security პრობლემა → `api-security-audit` ან `security-reviewer`
- Database/data integrity → `backend-architect`
- Error handling → `debugger`

### 2.4 — TDD (ჯერ ტესტი, შემდეგ კოდი)
```
1. failing test ❌  (regression prevention)
2. ფიქსი
3. ტესტი ✅
4. სხვა ტესტებიც ✅
```

### 2.5 — Timeout წესი
30 წუთში ვერ წყვეტ → tracker-ში `❌ დაბლოკილია` + მიზეზი → შემდეგ critical-ზე გადადი.

### 2.6 — დასრულება
```bash
$TEST_CMD
$LINT_CMD

# Tracker update
✅ $AUDIT_DIR/00_PROGRESS_TRACKER.md → ✅ შესრულდა
✅ $AUDIT_DIR/00_BEFORE_AFTER_METRICS.md
✅ $AUDIT_DIR/00_DECISIONS_LOG.md

git add .
git commit -m "fix: critical — [issue title]"
```

---

## ნაბიჯი 3 — სტატუსების flow

```
⬜ მოლოდინში   → 🔄 მიმდინარე   (იწყებ)
🔄 მიმდინარე   → ✅ შესრულდა    (ტესტი + commit)
🔄 მიმდინარე   → ❌ დაბლოკილია  (30 წთ+ ან გარე deps)
```

**მნიშვნელოვანი:**
- ნუ ფიქსავ High/Medium/Low — სხვა ფაზებშია
- ნუ შეეხები სხვა ფაილებს გარდა პირდაპირი კავშირისა
- გაუგებრობის შემთხვევაში — შეჩერდი და მკითხე

---

## სესიის ბოლოს ანგარიში

```markdown
## ✅ შესრულდა (Critical)
- [ფაილი + ხაზი]: ცვლილება + commit hash

## ❌ დაბლოკილია
- [ფაილი]: მიზეზი + რა სჭირდება გასაგრძელებლად

## ⚠️ ყურადღება
- [რისკები, regression შესაძლებლობები]

## 🤖 გამოყენებული agents
- [agent სახელი]: [რისთვის]
```

---

## შემდეგი ნაბიჯი

```
/audit:3b_verify_critical
```
