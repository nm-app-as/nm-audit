ერთი კატეგორიის სწრაფი audit. სრული pipeline-ის გარეშე.

გამოყენება:
- კონკრეტული ცვლილების შემდეგ
- PR merge-ამდე სწრაფი ანგარიში
- სრული pipeline არ გჭირდება

---

```bash
AUDIT_DIR="audit/$(cat audit/LATEST 2>/dev/null || echo "no-active")"
TEST_CMD=$(jq -r '.commands.test_coverage // .commands.test // "npm test"' audit/.config.json 2>/dev/null || echo "npm test")
LINT_CMD=$(jq -r '.commands.lint // "npm run lint"' audit/.config.json 2>/dev/null || echo "npm run lint")
```

**შენიშვნა:** quick_check-მა შესაძლოა იმუშაოს active audit-ის გარეშე. იდეალურად
ჯერ `/audit:0_init` და `/audit:1_audit` იმუშავოს, რომ summary-ი შენახული ფაილი არსებობდეს.

---

ჯერ — რომელი კატეგორია?

თუ user-მა მითითა → გამოიყენე.
თუ არა → კითხე:

"რომელი კატეგორია?

  1. security      — auth, inputs, secrets, headers, OWASP
  2. testing       — coverage, test quality
  3. performance   — slow queries, N+1, bundle size
  4. architecture  — coupling, module structure
  5. code_quality  — complexity, duplication, type safety
  6. cicd          — pipeline speed, environments
  7. database      — indexes, queries, schema
  8. documentation — README, API docs, onboarding"

---

## SECURITY check

**Agent რეკომენდაცია:** `security-reviewer` ან `api-security-audit` deep review-სთვის.

```bash
git log --all -p | grep -iE "(password|secret|api_key|token)=" | head -20
npm audit --audit-level=high
grep -r "helmet\|X-Frame-Options\|Content-Security-Policy" src/ --include="*.ts"
grep -rn "router\.\(get\|post\|put\|delete\|patch\)" src/ --include="*.ts" | head -30
```

შედეგი → `$AUDIT_DIR/00_SUMMARY.md` (თუ active audit არსებობს).

---

## TESTING check

```bash
$TEST_CMD

# Files without tests
find src -name "*.ts" ! -name "*.test.ts" ! -name "*.d.ts" | while read f; do
  base="${f%.ts}"
  [ ! -f "${base}.test.ts" ] && echo "NO TEST: $f"
done

# Tests without assertions
grep -rn "it\|test" src --include="*.test.ts" -A3 | grep -v "expect\|assert" | head -20
```

---

## PERFORMANCE check

```bash
# findMany without where (potential full-table scan)
grep -rn "findMany()" src --include="*.ts" | head -20

# Synchronous blocking
grep -rn "readFileSync\|writeFileSync\|execSync" src --include="*.ts"
```

### N+1 detection

ცრუ-positive-ით სავსე grep-ი არ ვიყენებთ. ნაცვლად:

გამოიყენე `backend-architect` ან `architect-reviewer` agent ძირითად service/controller ფაილებზე.
agent-მა უნდა ეძებოს კონკრეტულად: loop შიგნით `await find/findMany`, controller რომელიც DB-ზე
ერთი response-ისთვის რამდენჯერმე იძახებს.

შედეგი — ფაილების სიად, ცარიელი count არა.

---

## ARCHITECTURE check

```bash
grep -rn "from '../../" src --include="*.ts" | wc -l
grep -rn "from '../../../" src --include="*.ts" | wc -l
find src -name "*.ts" ! -name "*.test.ts" | xargs wc -l | sort -rn | head -20
npx madge --circular src/
```

---

## CODE_QUALITY check

```bash
$LINT_CMD
grep -rn ": any\|as any" src --include="*.ts" | grep -v ".test.ts" | wc -l
npx ts-prune 2>/dev/null | head -30
```

---

## CICD check

```bash
ls .github/workflows/
cat .github/workflows/*.yml | grep -E "test|lint|build"
cat .github/workflows/*.yml | grep -E "staging|production|preview"
gh run list --limit 5 --json durationMs,conclusion,name 2>/dev/null
```

---

## DATABASE check

```bash
# Missing indexes (Prisma)
cat prisma/schema.prisma 2>/dev/null | grep -A10 "model " | grep -v "@@index\|@id\|@unique" | head -40

# findMany without pagination
grep -rn "findMany()" src --include="*.ts" | grep -v "take\|limit\|skip" | head -20

# Raw SQL
grep -rn "\$queryRaw\|\$executeRaw\|query(" src --include="*.ts" | head -20

# SSL
grep -rn "ssl\|sslmode" src --include="*.ts" .env.example 2>/dev/null | head -10
```

---

## DOCUMENTATION check

```bash
grep -iE "install|setup|run|test|deploy|env|config" README.md | wc -l

diff <(grep "^[A-Z]" .env.example 2>/dev/null | cut -d= -f1 | sort) \
     <(grep "process.env\." src -rh --include="*.ts" | grep -oE "[A-Z_]+" | sort -u)

grep -rn "router\.\(get\|post\|put\|delete\)" src --include="*.ts" -B1 | grep -v "\/\/" | head -20
```

---

## OUTPUT FORMAT (ნებისმიერი კატეგორიისთვის)

```
## Quick Check — [category] — [date]

### Score: X.X / 10

### What was checked
- [command 1]: [result]
- [command 2]: [result]

### Issues found
🔴 Critical: [list]
🟡 Medium: [list]
🟢 Low/Info: [list]

### Compared to last
Last ([date]): X.X → Now: X.X  ([+/-] X.X)

### Recommended action
[fix now / monitor / no action needed]
```
