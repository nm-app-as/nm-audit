ეს ფაილი იშვება, როცა აუდიტის ციკლი გასაუქმებელია — main-ს ვუბრუნდებით უსაფრთხოდ.

⚠️ destructive ბრძანებებს ეს ფაზა არ აკეთებს — branch რჩება, რომ user-მ თვითონ
   გადაწყვიტოს როდის წაშალოს. audit/$DATE/ ფოლდერი ხელუხლებელია.

---

## ნაბიჯი 0 — სტატუსი

```bash
echo "მიმდინარე branch:"
git branch --show-current

echo ""
echo "untracked / modified ფაილები:"
git status --short

echo ""
echo "active audit:"
[ -f audit/LATEST ] && cat audit/LATEST || echo "(LATEST pointer არ არის)"
```

---

## ნაბიჯი 1 — uncommitted ცვლილებების სტეშირება

```bash
if [ -n "$(git status --porcelain)" ]; then
  STASH_MSG="abort-audit-$(date +%Y-%m-%d-%H%M)"
  git stash push -u -m "$STASH_MSG"
  echo "✅ ცვლილებები შენახული stash-ში: $STASH_MSG"
  echo "სიის სანახავად: git stash list"
  echo "დასაბრუნებლად: git stash pop"
fi
```

`-u` flag — untracked ფაილებიც ჩავიდეს stash-ში.

---

## ნაბიჯი 2 — main-ზე გადასვლა

```bash
git checkout main
```

თუ main-ზე გადასვლა ვერ მოხერხდა (conflict, dirty tree) — გაჩერდი და მოახსენე user-ს.
არ გააკეთო `git checkout -f` ან სხვა force ბრძანება — ჯერ user-ს გადააწყვეტინე.

---

## ნაბიჯი 3 — შეტყობინება user-ს

```
✅ ვუბრუნდი main branch-ს.

რა დარჩა ხელუხლებელი:
  - audit branch (ნახვა: git branch | grep audit/)
  - audit/$(cat audit/LATEST 2>/dev/null)/ ფოლდერი
  - stash (თუ იყო ცვლილებები — `git stash list`)

თუ გინდა აუდიტის სრული გაწმენდა (destructive — გადადი მხოლოდ მაშინ, როცა დარწმუნებული ხარ):
  git branch -D audit/<date>          # branch წაშლა
  rm -rf audit/<date>                 # output ფოლდერი წაშლა
  git stash drop stash@{0}            # stash წაშლა

თუ გინდა დაბრუნება იმავე ციკლზე — git checkout audit/<date>.
```
