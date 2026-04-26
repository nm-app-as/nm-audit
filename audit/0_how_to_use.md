# Audit Pipeline — გამოყენების ინსტრუქცია

ეს ფოლდერი შეიცავს 7 აუდიტის ფაზას + 2 დამხმარე ფაილს (`init`, `abort`) + `quick_check`-ს.
ფაზები ერთიმეორის შემდეგ ეშვება. ყოველი ფაზა აგრძელებს წინას.

---

## პირველად — ერთხელ პროექტში

```
init                ← აღმოაჩენს stack-ს, ქმნის audit/.config.json
```

`init` ერთხელ ეშვება პროექტში. ყველა მომდევნო ფაზა მისი output-ის (`audit/.config.json`)
წყალობით იცნობს test/lint/build commands-ს, deploy platform-ს და monorepo სტრუქტურას.
თუ stack იცვალა — წაშალე `audit/.config.json` და ხელახლა გაუშვი.

---

## სრული Pipeline (პირველი audit ან major release)

```
1_audit              ← სრული სკანი, ქმნის audit/YYYY-MM-DD/
2_after_audit        ← output-ი სტრუქტურირებულ ფაილებში გადაიტანება

3_fix_critical       ← 🔴 Critical issues (CVSS 9+)
3b_verify_critical   ← Security / Error Handling / Database re-score

3_fix_high           ← 🟡 High issues (CVSS 7-8)
3b_verify_high       ← Testing / Architecture re-score

3_fix_medium         ← 🟢 Medium / Refactoring (approval-ით)
3b_verify_medium     ← Code Quality / Documentation re-score

4_run_project        ← runtime checks: endpoints, load test, deeper scan
5_attack             ← red team / blue team security
6_architecture_evolution  ← მომდევნო scale-ის არქიტექტურის გეგმა
7_final              ← investment verdict + HTML dashboard
```

დრო: მძიმე. გამოყენება: pre-launch, ყოველკვარტალური review, major refactor.

---

## სწრაფი Pipeline (PR-ის შემდეგ, ყოველკვირეული)

```
quick_check          ← ერთი კატეგორია: security / testing / performance / ...
3b_verify_[severity] ← regression-ის შემოწმება
```

დრო: მსუბუქი. გამოყენება: PR merge-ის შემდეგ, bug fix-ის შემდეგ.

---

## ინკრემენტული Pipeline (sprint-ის ბოლოს)

```
3_fix_high           ← outstanding high issues
3b_verify_high       ← ქულების გადამოწმება
5_attack             ← მხოლოდ თუ auth/API შეიცვალა
7_final              ← HTML dashboard-ის განახლება
```

---

## თუ რამე ცუდად წავიდა

```
abort                ← უსაფრთხოდ უბრუნდება main-ს, არ შლის audit branch-ს ან output-ს
```

destructive არაფერი — branch რჩება, user თვითონ წყვეტს როდის წაშალოს.

---

## Branch ლოგიკა (მნიშვნელოვანი)

ერთი audit ციკლზე — **ერთი** branch (`audit/YYYY-MM-DD`).

- `1_audit` ქმნის ახალ branch-ს და ფოლდერს.
- ყველა მომდევნო ფაზა (`2_after_audit`-დან `7_final`-მდე) **იმავე** branch-ზე რჩება.
- Branch-ი არ იცვლება. ყველა ფიქსი იქ ხვდება.
- main ხელუხლებელი რჩება სანამ user-ი ხელით (ან PR-ით) არ შემოიტანს.

---

## ფოლდერის სტრუქტურა (ქმნის 1_audit + 2_after_audit)

```
audit/
├── .config.json               ← stack/commands info (init ქმნის ერთხელ)
├── LATEST                     ← "2026-04-26" (active audit-ის pointer)
├── CHANGELOG.md               ← ყველა audit ციკლის overall score + verdict
├── 2026-04-26/                ← თარიღით — ყოველი audit ცალკე
│   ├── report/
│   │   └── index.html         ← 7_final ქმნის
│   ├── 00_SUMMARY.md
│   ├── 00_PROGRESS_TRACKER.md
│   ├── 00_BEFORE_AFTER_METRICS.md
│   ├── 00_REGRESSION_RISKS.md
│   ├── 00_DECISIONS_LOG.md
│   ├── 01_architecture/
│   ├── 02_code_quality/
│   ├── 03_testing/
│   ├── 04_cicd/
│   ├── 05_git_github/
│   ├── 06_security/
│   ├── 07_performance/
│   ├── 08_documentation/
│   └── 09_automated_checks/
└── 2026-07-15/                ← მომდევნო audit ცალკე
```

ყოველი ფაზა კითხულობს: `AUDIT_DIR="audit/$(cat audit/LATEST)"`.

---

## რომელი ფაზა — როდის

| სიტუაცია | გამოყენება |
|-----------|-----|
| ახალი პროექტი (პირველი ციკლი) | init → 1 → 2 → 3_fix_critical → 3b → 3_fix_high → 3b → 3_fix_medium → 3b → 4 → 5 → 6 → 7 |
| Bug fix-ის შემდეგ | 3_fix_high → 3b_verify_high → 7 |
| ახალი API endpoint | quick_check security → 5 → 3b_verify_critical |
| Pre-launch | სრული Pipeline |
| "უსაფრთხოება ხო კარგადაა?" | quick_check security |
| "Refactor-მა ხომ არ გატეხა?" | 3b_verify_medium |
| Quarterly review | 4 → 5 → 6 → 7 |
| რამე გაფუჭდა | abort |
