შენ ხარ Technical Writer + Project Manager.
1_audit-ის შედეგები მზადაა. მოაწყე სტრუქტურირებული ფაილების სახით.

---

## ნაბიჯი 0 — კონტექსტი

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"
echo "Active audit: $AUDIT_DIR"
git branch --show-current
```

დარწმუნდი რომ იგივე branch-ზე ვართ, რაც 1_audit-მა შექმნა.

---

## ნაბიჯი 1 — ფოლდერ სტრუქტურა

```bash
mkdir -p "$AUDIT_DIR/report"
mkdir -p "$AUDIT_DIR/01_architecture"
mkdir -p "$AUDIT_DIR/02_code_quality"
mkdir -p "$AUDIT_DIR/03_testing"
mkdir -p "$AUDIT_DIR/04_cicd"
mkdir -p "$AUDIT_DIR/05_git_github"
mkdir -p "$AUDIT_DIR/06_security"
mkdir -p "$AUDIT_DIR/07_performance"
mkdir -p "$AUDIT_DIR/08_documentation"
mkdir -p "$AUDIT_DIR/09_automated_checks"
```

---

## ნაბიჯი 2 — root ფაილები

### `$AUDIT_DIR/00_SUMMARY.md`
- Overall score + verdict
- Top 3 critical
- Top 3 strengths
- Next steps (1 week)

### `$AUDIT_DIR/00_PROGRESS_TRACKER.md`

```markdown
| # | კატეგორია | პრობლემა | სიმძიმე | სტატუსი | შენიშვნა |
|---|-----------|----------|---------|---------|----------|
| 1 | Security  | SQL injection in /api/users | 🔴 | ⬜ მოლოდინში | |
| 2 | Testing   | 0% coverage in payment module | 🔴 | ⬜ მოლოდინში | |
```

სტატუსები: `⬜ მოლოდინში` / `🔄 მიმდინარე` / `✅ შესრულდა` / `❌ დაბლოკილია`

### `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md`

```markdown
| მეტრიკა | აუდიტამდე | ამჟამად | მიზანი |
|---------|-----------|---------|--------|
| Test coverage | X% | — | 80% |
| Build time | Xm Xs | — | <2m |
| npm vulnerabilities | X | — | 0 |
| TypeScript `any` count | X | — | 0 |
| Files > 400 lines | X | — | 0 |
| Cyclomatic complexity (avg) | X | — | <10 |
```

### `$AUDIT_DIR/00_REGRESSION_RISKS.md`

```markdown
## რისკი #1
- 🔧 ცვლილება: [მოდული/ფაილი]
- ⚠️ რისკი: [რა შეიძლება დაზარალდეს]
- 🛡 დაცვა: [რომელი ტესტები უნდა გაეშვას]
- 📍 კრიტიკული ფაილები: [src/...]
```

### `$AUDIT_DIR/00_DECISIONS_LOG.md`

ADR ფორმატი:

```markdown
## ADR-001: [სათაური]
- 📅 თარიღი: YYYY-MM-DD
- 🔍 კონტექსტი: [რატომ გახდა საჭირო]
- ✅ გადაწყვეტილება: [რა გავაკეთეთ]
- 🔄 ალტერნატივები: [რა განვიხილეთ]
- 📊 მოსალოდნელი შედეგი: [რა გვეგულება]
```

---

## ნაბიჯი 3 — კატეგორიის ფაილები (მხოლოდ 2 თითოეულში)

თითოეულ კატეგორიის ფოლდერში მხოლოდ ეს ორი ფაილი:

### `ISSUES.md`
```markdown
## პრობლემა #N
- 📍 ფაილი: src/..., ხაზი: N
- 🔴 სიმძიმე: კრიტიკული / 🟡 საშუალო / 🟢 დაბალი
- ❌ პრობლემა: [რა და რატომ]
- ✅ გამოსწორება: [ზუსტად რა შეიცვალოს]
- 💻 კოდის მაგალითი: before → after
- ⚠️ რეგრესიის რისკი:
- ⏱ სავარაუდო დრო: X საათი
```

### `FIXES.md`
copy-paste ready კოდი.

**ცარიელი template ფაილები არ ქმნი** (PIPELINE_CONFIG.md, ESLINT_PRETTIER_CONFIG.md,
README_TEMPLATE.md, SECURITY_CHECKLIST.md და ა.შ.). on-demand შეიქმნება, თუ რეალურად დაგვჭირდება.

---

## ნაბიჯი 4 — ROADMAP

`$AUDIT_DIR/ROADMAP.md`:

```markdown
## 🔴 Phase 1 — კრიტიკული (1-3 დღე)
[Security და Data Integrity პრობლემები]

## 🟡 Phase 2 — მნიშვნელოვანი (1-2 კვირა)
[High severity პრობლემები]

## 🟢 Phase 3 — გაუმჯობესება (1 თვე)
[Medium პრობლემები, refactoring]

## ⚪ Phase 4 — სამომავლო
[Low priority, nice-to-have]
```

---

## მოთხოვნები

- კომენტარები ქართულად, კოდი ინგლისურად
- კოდის მაგალითები copy-paste მზა
- არ გამოტოვო პრობლემა audit-დან
- PROGRESS_TRACKER ავტომატურად შეივსოს ყველა issue-თი
- BEFORE_AFTER_METRICS-ში რეალური მნიშვნელობები პროექტიდან

---

## შემდეგი ნაბიჯი

```
/audit:3_fix_critical
```
