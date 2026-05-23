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
git commit --allow-empty -m "$(cat <<'EOF'
checkpoint: before fixing critical issues

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
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

### 2.2 — ტრიაჟი
- 🔴 Critical: ფიქსავ დაუყოვნებლივ
- 🟡 High ან 🟢 Medium: ეს ფაზა არ ეკუთვნი — გამოტოვე

CVSS რეიტინგი მხოლოდ **Security** კატეგორიის issue-ებზე გამოიყენე (9+ = Critical, 7-8 = High).
დანარჩენ კატეგორიებზე (Testing, Architecture, Error Handling, DB...) — 🔴/🟡/🟢 emoji-ის მიხედვით იხელმძღვანელე, რომელიც `1_audit`-მა მიანიჭა.

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

ჯერ ტესტი + lint — თუ ერთერთი ვარდება, commit ბლოკდება:

```bash
$TEST_CMD && $LINT_CMD || { echo "❌ tests/lint failed — commit ბლოკდება. გაასწორე ჯერ."; exit 1; }
```

tracker-ის განახლება (ხელით edit, არა bash):
- `$AUDIT_DIR/00_PROGRESS_TRACKER.md` → ✅ შესრულდა
- `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`
- `$AUDIT_DIR/00_DECISIONS_LOG.md`

commit Co-Authored-By trailer-ით:

```bash
git add .
git commit -m "$(cat <<'EOF'
fix: critical — [issue title]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
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
