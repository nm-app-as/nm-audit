შენ ხარ Senior Auditor. ჩაატარე ამ პროექტის სრული, უკომპრომისო აუდიტი.
გამოიყენე specialized agents პარალელურად კატეგორიებზე.

---

## ფაზა 0 — Config + Git Safety + Audit Folder

### 0.1 — Config-ის წინასწარი მოთხოვნა

```bash
if [ ! -f audit/.config.json ]; then
  echo "❌ audit/.config.json არ არის."
  echo "ჯერ გაუშვი: /audit:0_init"
  exit 1
fi

# ჩატვირთე commands
TEST_CMD=$(jq -r '.commands.test_coverage // .commands.test // "npm test"' audit/.config.json)
LINT_CMD=$(jq -r '.commands.lint // "npm run lint"' audit/.config.json)
TYPECHECK_CMD=$(jq -r '.commands.typecheck // "npx tsc --noEmit"' audit/.config.json)
PLATFORM=$(jq -r '.deploy_platform // "unknown"' audit/.config.json)
GITNEXUS=$(jq -r '.gitnexus_indexed // false' audit/.config.json)
echo "Stack ready: test='$TEST_CMD', lint='$LINT_CMD', platform=$PLATFORM, gitnexus=$GITNEXUS"
```

თუ `GITNEXUS=true` — Architecture, Security, Error Handling ფაზებში გამოიყენე `gitnexus_*` MCP tools (impact / query / cypher) grep-ის და madge-ის ნაცვლად. ეს უფრო სანდო შედეგს იძლევა (false-positive-ი ნაკლები) და უკვე ინდექსირებული dep graph-ი გვაქვს.

### 0.2 — Git safety + ერთი branch ციკლზე

```bash
# შეინახე მიმდინარე სამუშაო
git stash push -u -m "before-audit-$(date +%Y-%m-%d)" || true

# ერთი ციკლი — ერთი branch
AUDIT_DATE=$(date +%Y-%m-%d)
BRANCH="audit/$AUDIT_DATE"

if git rev-parse --verify "$BRANCH" >/dev/null 2>&1; then
  git checkout "$BRANCH"
else
  git checkout -b "$BRANCH"
fi

echo "Active branch: $(git branch --show-current)"
```

### 0.3 — Audit folder

```bash
AUDIT_DIR="audit/$AUDIT_DATE"
mkdir -p "$AUDIT_DIR/report"
echo "$AUDIT_DATE" > audit/LATEST
```

ყველა output → `audit/$AUDIT_DATE/`. ძველი audit-ები ხელუხლებლები.
ყველა ფიქსი → `audit/$AUDIT_DATE` branch-ზე. main ხელუხლებელი.
**მომდევნო ფაზები არ ქმნიან ახალ branch-ს — ყველა ერთ branch-ზე რჩება.**

---

## ფაზა 1 — სრული სკანირება (პარალელურად)

გამოიყენე specialized agents შესაბამის კატეგორიებზე — ყველა ერთ message-ში
ერთდროული tool-call-ებით, რომ pipe-ი პარალელურად მუშაობდეს.

### 🔒 უსაფრთხოება — 20%
**Agents:** `security-reviewer`, `api-security-audit`
- OWASP Top 10 ყველა endpoint-ზე
- Hardcoded secrets: `git log --all --diff-filter=A -p -- '*.env' '*.key' '*.pem' | head -50`
- Dependency CVEs: `npm audit --audit-level=high`
- Input validation (Zod ან სხვა schema)
- Auth middleware ყოველ protected route-ზე
- Security headers (helmet ან მსგავსი)

თუ GitNexus indexed:
- `gitnexus_query({query: "auth middleware authentication"})` — auth flow-ების სრული რუკისთვის
- `gitnexus_query({query: "input validation user input"})` — boundary-ების სიისთვის
- `gitnexus_context({name: "<endpoint handler>"})` — ყოველი endpoint-ის სრული callers/callees

### 📦 Code Quality — 15%
**Agent:** `code-reviewer`
- Readability, DRY/SOLID/KISS
- ფუნქციები > 50 ხაზი
- Dead code, unused imports
- TypeScript `any`: `grep -rn ": any\|as any" src --include="*.ts" | wc -l`

### 🧪 Testing — 15%
**Agent:** `test-engineer`
- გამოიყენე `$TEST_CMD` (config-დან)
- Unit / integration / e2e existence
- Files without tests
- Edge case coverage

### 🏗 Architecture — 10%
**Agent:** `architect-reviewer`
- Dependency graph — ყველაზე მეტი inbound dep რომელ module-ს?
- Cross-module imports: `grep -rn "from '../../" src --include="*.ts" | wc -l`
- Circular deps: `npx madge --circular src/`
- 10x scale-ზე რა გატყდება?

თუ GitNexus indexed (აჯობებს grep + madge-ს):
- `gitnexus_cypher({query: "MATCH (n)<-[r:CALLS|IMPORTS]-(m) RETURN n.name, count(m) AS deps ORDER BY deps DESC LIMIT 10"})` — ყველაზე "load-bearing" symbols
- `gitnexus_query({query: "module coupling cross-cutting"})` — cohesion gap-ები
- Circular deps GitNexus-ის graph-დან — უფრო ზუსტი vs madge

### ⚠️ Error Handling — 10%
**Agents:** `debugger`, `code-reviewer`
- Unhandled promise rejections
- ცარიელი catch blocks: `grep -rn "catch.*{}" src --include="*.ts"`
- Production stack traces?
- DB/external API down — graceful degradation?

თუ GitNexus indexed:
- `gitnexus_query({query: "error handling exception catch"})` — ყველა error path
- `gitnexus_context({name: "errorHandler"})` ან მსგავსი — error middleware-ის callers

### 🖥 Frontend — 10%
**Agent:** `frontend-developer` (თუ frontend არსებობს)
- Bundle size, Lighthouse
- Accessibility: alt, headings, keyboard nav
- Mobile responsiveness
- თუ frontend არ არის — ქულა N/A, წონა გადანაწილდეს Security +5% / Testing +5%

### ⚙️ CI/CD — 5%
**Agent:** `general-purpose`
- `ls .github/workflows/` — pipeline არსებობს?
- ტესტები CI-ში?
- Staging vs Production გამიჯვნა
- Rollback strategy

### 🗄 Database — 5%
**Agent:** `backend-architect`
- FK indexes
- Pagination-ის გარეშე queries
- Raw SQL (injection risk)
- SSL connection?

### 🔀 Git — 5%
**Agent:** `general-purpose`
- Branching strategy
- Commit message quality (`git log --oneline -20`)
- Bus factor: `git log --format='%an' | sort | uniq -c | sort -rn`

### 📖 Documentation — 5%
**Agent:** `document-specialist`
- README: setup/run/test/deploy
- `.env.example` up-to-date
- API endpoints documented
- Onboarding time estimate

---

## ფაზა 2 — შეფასება

იყავი **უკიდურესად კრიტიკული და პირდაპირი**.
დაასახელე კონკრეტული ფაილები, ხაზები, პრობლემები.
დაწერე ისე, თითქოს შენი რეპუტაცია ამ შეფასებაზეა დამოკიდებული.

### ქულების ფორმულა

```
overall = (security×0.20) + (code_quality×0.15) + (testing×0.15)
        + (architecture×0.10) + (error_handling×0.10) + (frontend×0.10)
        + (cicd×0.05) + (database×0.05) + (git×0.05) + (documentation×0.05)
```

თუ Frontend არ არსებობს: 10% გადანაწილდეს Security +5% და Testing +5%-ზე
(ფორმულა: security 25%, testing 20%, frontend 0%, დანარჩენი იგივე).

ქულის მინიჭების კონკრეტული რუბრიკა → `/audit:0_rubric` (0-2 = კატასტროფული, 9-10 = წარჩინებული).

### Output ფორმატი

```
## Audit Report — [date]

### Overall Score: X.X / 10

| კატეგორია       | ქულა | წონა | Issues |
|-----------------|------|------|--------|
| Security        | X.X  | 20%  | 🔴 N |
| Code Quality    | X.X  | 15%  | 🟡 N |
| Testing         | X.X  | 15%  | coverage: X% |
| Architecture    | X.X  | 10%  | |
| Error Handling  | X.X  | 10%  | |
| Frontend        | X.X  | 10%  | |
| CI/CD           | X.X  |  5%  | |
| Database        | X.X  |  5%  | |
| Git             | X.X  |  5%  | |
| Documentation   | X.X  |  5%  | |

### 🔴 Critical (immediate)
[file:line refs]

### 🟡 High (this sprint)

### 🟢 Medium/Low (later)

### Metrics
- Test coverage: X%
- npm vulnerabilities: X high, X critical
- TypeScript `any`: X
- Files > 400 lines: X
```

---

## ფაზა 3 — შენახვა

ქულები + შეჯამება პირდაპირ `$AUDIT_DIR/00_SUMMARY.md`-ში.

### CHANGELOG ჩანაწერი

```bash
# შექმენი თუ არ არსებობს
if [ ! -f audit/CHANGELOG.md ]; then
  cat > audit/CHANGELOG.md <<'EOF'
# Audit Score History

| Date | Overall | Verdict |
|------|---------|---------|
EOF
fi

# ერთი ხაზი დაამატე (verdict 7_final-ში დასრულდება)
echo "| $AUDIT_DATE | X.X | (in progress) |" >> audit/CHANGELOG.md
```

---

## შემდეგი ნაბიჯი

```
/audit:2_after_audit
```
