პროექტში მიმდინარე აუდიტის სტატუსი. რომელ ფაზაზე ვართ, რა გადარჩა, რომელი ბრძანება გავუშვა შემდეგი.

---

## ნაბიჯი 0 — აქტიური აუდიტი არსებობს?

```bash
if [ ! -f audit/LATEST ]; then
  echo "❌ active audit-ი არ არის."
  echo ""
  echo "ჯერ გაუშვი:"
  echo "  /audit:0_init       (თუ ჯერ არ ხარ run-ი ამ პროექტში)"
  echo "  /audit:1_audit    (ახალი ციკლის დასაწყებად)"
  exit 0
fi

AUDIT_DATE=$(cat audit/LATEST)
AUDIT_DIR="audit/$AUDIT_DATE"
BRANCH="audit/$AUDIT_DATE"

echo "📅 Audit ციკლი: $AUDIT_DATE"
echo "📁 Folder:     $AUDIT_DIR"
echo "🌿 Branch:     $BRANCH"
echo ""
```

---

## ნაბიჯი 1 — Branch + ბოლო commit-ები

```bash
CURRENT=$(git branch --show-current)
echo "მიმდინარე branch: $CURRENT"

if [ "$CURRENT" != "$BRANCH" ]; then
  echo "⚠️ სხვა branch-ზე ხარ. გადასვლა: git checkout $BRANCH"
fi

echo ""
echo "ბოლო 10 commit ამ branch-ზე:"
git log --oneline -10
```

---

## ნაბიჯი 2 — PROGRESS_TRACKER-ის ანალიზი

```bash
if [ ! -f "$AUDIT_DIR/00_PROGRESS_TRACKER.md" ]; then
  echo "❌ PROGRESS_TRACKER.md არ არსებობს."
  echo "→ შემდეგი ნაბიჯი: /audit:2_after_audit"
  exit 0
fi

cat "$AUDIT_DIR/00_PROGRESS_TRACKER.md"
```

ცხრილი წაიკითხე და დათვალე:
- 🔴 Critical: სკოპის `✅ შესრულდა` / `⬜ მოლოდინში` / `🔄 მიმდინარე` / `❌ დაბლოკილია`
- 🟡 High: იგივე
- 🟢 Medium/Low: იგივე

---

## ნაბიჯი 3 — რომელი ფაზები გავიდა?

```bash
echo "ფაზების კვალი commit-ებიდან:"
git log --oneline | grep -iE "fix: critical|fix: high|refactor:|docs:|checkpoint:" | head -20

echo ""
echo "Report HTML არსებობს?"
[ -f "$AUDIT_DIR/report/index.html" ] && echo "✅ Yes — 7_final გავიდა" || echo "❌ No"
```

---

## ნაბიჯი 4 — შემდეგი ფაზის რეკომენდაცია

ლოგიკა:

| მდგომარეობა | შემდეგი ფაზა |
|-------------|-------------|
| `00_PROGRESS_TRACKER.md` არ არსებობს | `/audit:2_after_audit` |
| 🔴 outstanding > 0 | `/audit:3_fix_critical` |
| 🔴 ყველა done, verify არ გავლილა (ბოლო commit `fix: critical`) | `/audit:3b_verify_critical` |
| 🔴 verified, 🟡 outstanding > 0 | `/audit:3_fix_high` |
| 🟡 ყველა done, verify არ გავლილა | `/audit:3b_verify_high` |
| 🟡 verified, 🟢 outstanding > 0 | `/audit:3_fix_medium` |
| 🟢 done, verify არ გავლილა | `/audit:3b_verify_medium` |
| ყველა severity verified, runtime ფაზა არ გავლილა | `/audit:4_run_project` |
| 4 done, attack ფაზა არ გავლილა | `/audit:5_attack` |
| 5 done, architecture ფაზა არ გავლილა | `/audit:6_architecture_evolution` |
| 6 done, final ფაზა არ გავლილა | `/audit:7_final` |
| `report/index.html` არსებობს | აუდიტი დასრულდა — PR / merge / abort |

---

## OUTPUT ფორმატი

```
## Audit Status — [date]

📅 Cycle:  audit/[date]
🌿 Branch: [current] [✅ on track / ⚠️ checkout needed]
🔧 Last:   [hash] [short message]

### Progress
🔴 Critical:    [X done / Y outstanding / Z blocked]
🟡 High:        [X / Y / Z]
🟢 Medium/Low:  [X / Y / Z]

### Phases passed (commit kvali)
✅ 1_audit              — [date]
✅ 2_after_audit        — [date]
✅ 3_fix_critical       — [date or "—"]
✅ 3b_verify_critical   — [date or "—"]
🔄 3_fix_high           — running
⬜ 3b_verify_high       — pending
⬜ 3_fix_medium
⬜ 3b_verify_medium
⬜ 4_run_project
⬜ 5_attack
⬜ 6_architecture_evolution
⬜ 7_final

### Next step
→ /audit:[recommended phase]

### Notes
[რა საჭიროებს ყურადღებას — blocked issues, regression flags, branch mismatch]
```

---

## წესები

- `status` არ ცვლის ფაილებს, არ committ-ავს, არ მართავს test-ებს.
- მხოლოდ კითხულობს state-ს და გვაძლევს რეკომენდაციას.
- თუ ვერ მიხვდი რომელ ფაზაზე ვართ — გადახედე commit history-ს და PROGRESS_TRACKER-ს.
