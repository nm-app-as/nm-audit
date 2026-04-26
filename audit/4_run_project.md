პროექტის runtime / deeper აუდიტი — ის ნაწილები, რასაც სტატიკურმა scan-მა გამოტოვა.
გამოიყენე specialized agents პარალელურად.

---

## ფაზა 0 — კონტექსტი (იგივე branch რჩება)

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"
echo "Active audit: $AUDIT_DIR"

# ⚠️ ახალ branch-ს არ ვქმნით — 1_audit-მა უკვე შექმნა და ყველა ფიქსი იქ მიდის.
git branch --show-current
```

---

## ფაზა 1 — Runtime Behavior

**Agents (პარალელურად):** `qa-tester`, `api-security-audit`, `backend-architect`, `general-purpose`.

ყურადღება იმაზე, რასაც სტატიკური scan-ი ვერ აღმოაჩენს:

- **Endpoint behavior**: გაუშვი server, hit ყველა API endpoint, ჩაწერე response time
- **Auth bypass**: სცადე anonymous → user → admin escalation
- **Dependency CVEs**: `npm audit --audit-level=high`
- **Secrets in git history**:
  ```bash
  git log --all -p | grep -iE "(password|secret|api_key|token)=" | head -20
  ```
- **Database**: `EXPLAIN` critical queries, N+1, missing indexes
- **Bundle**: size, tree-shaking, code splitting (frontend)
- **Memory leaks**: event listener cleanup, React subscription management
- **Race conditions**: concurrent mutation points API route-ებში
- **Error recovery**: DB down → ?, third-party down → ?, payment timeout → ?

---

## ფაზა 2 — REPORT FINDINGS, არ ფიქსო (Triage-ის გარეშე)

ახალი issues → ჩაწერე `00_PROGRESS_TRACKER.md`-ში. Triage:

- **CVSS 9+ (Critical)**: ფიქსი დაუყოვნებლივ, commit, report
- **CVSS 7-8 (High)**: ფიქსი — ფაილების სია commit-ამდე
- **CVSS 4-6 (Medium) ან დაბალი**: მხოლოდ document, არ ფიქსო approval-ის გარეშე

ეს ხელს უშლის 50+ ფაილის ჩუმად შეცვლას.

---

## ფაზა 3 — Score

გამოიყენე weighted scoring:
- Security 20%, Code Quality 15%, Testing 15%
- Architecture 10%, Error Handling 10%, Frontend 10%
- CI/CD 5%, Database 5%, Git 5%, Documentation 5%

BEFORE vs AFTER ცხრილი თითოეულ კატეგორიაზე.
**იყავი პატიოსანი — რეალურად გავაუმჯობესეთ თუ მხოლოდ ციფრებს ვამოძრავებთ?**

შედეგი → `$AUDIT_DIR/00_SUMMARY.md`.

---

## შემდეგი ნაბიჯი

```
/audit:5_attack
```
