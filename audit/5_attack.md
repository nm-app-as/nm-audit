PRODUCTION READINESS CERTIFICATION — red team / blue team adversarial audit.
პროექტმა უკვე გაიარა 3 წინა ფაზა.

---

## ფაზა 0 — კონტექსტი

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"

# წაიკითხე წინა შედეგები — არ აღმოაჩინო უკვე ცნობილი issues
cat "$AUDIT_DIR/00_PROGRESS_TRACKER.md"

# ⚠️ იგივე branch — 1_audit-ის შექმნილი audit/$DATE
git branch --show-current
```

---

## SCOPE BOUNDARIES (Red Team-მა უნდა დაიცვას)

- ❌ Real payment requests არ გააგზავნო — mock/intercept
- ❌ Production database არ flood-ო — test environment
- ❌ Real emails/notifications testing-ის დროს არ
- ⚠️ თუ არ ხარ დარწმუნებული destructive-ია თუ არა → STOP, კითხე

---

## RED TEAM (Attack)

**Agents:** `api-security-audit`, `security-reviewer`.

- OWASP Top 10 ყველა endpoint-ზე
- Privilege escalation: anonymous → user → admin
- Payment manipulation (price tampering, replay attacks)
- Data exfiltration error messages-ით ან timing attacks
- Rate limit bypass (IP spoofing, distributed)
- File upload exploits (polyglot, path traversal, XXE)
- SSRF URL-accepting parameters-ით
- JWT manipulation (algorithm confusion, token reuse)

დამატებითი:
- Token storage: JWT localStorage-ში (ცუდი) vs httpOnly cookies (კარგი)?
- `process.env` error messages/logs-ში გადის?
- Dependency confusion: internal package names hijack-ისთვის
- HTTP method override: PATCH/DELETE მუშაობს POST-ით + `_method`?

---

## BLUE TEAM (Defend)

**Agents:** `security-reviewer`, `executor`.

ყოველი წარმატებული attack-ისთვის → fix immediately.
- WAF rules, CSP headers, security headers
- Audit logging security events-ისთვის
- Intrusion detection patterns
- Incident response runbook

Triage:
- **CVSS 9+ (Critical)**: ფიქსი immediately, commit, report
- **CVSS 7-8 (High)**: ფიქსი — ფაილების სია commit-ამდე
- **CVSS 4-6 (Medium) ქვემოთ**: document only, fix-ი approval-ის გარეშე არ

---

## INFRASTRUCTURE AUDIT

```bash
# Platform — config-დან ან auto-detect
PLATFORM=$(jq -r '.deploy_platform // "unknown"' audit/.config.json)

if [ "$PLATFORM" = "unknown" ]; then
  [ -f railway.json ] || [ -f railway.toml ] && PLATFORM="Railway"
  [ -f vercel.json ] && PLATFORM="Vercel"
  [ -f fly.toml ] && PLATFORM="Fly.io"
  [ -f render.yaml ] && PLATFORM="Render"
  [ -f app.yaml ] && PLATFORM="Google App Engine"
  [ -f Procfile ] && PLATFORM="Heroku"
fi
echo "Detected platform: $PLATFORM"
```

შემოწმე platform-ის შესაბამისად:
- Deploy platform config (headers, env vars, deploy settings)
- DNS, SSL/TLS configuration
- DB connection security (SSL, connection pooling)
- Third-party integration security

---

## LOAD TEST SIMULATION

რომელ კონცენტრაციაში ცვივა — 100, 1000, 10000 concurrent?
რომელი component ცვივა პირველი?

```bash
# k6 ან artillery — load simulation, არა real traffic
k6 run --vus 100 --duration 30s load-test.js
```

ჩაწერე: რომელი endpoint ცვივა პირველი, რა error, რა concurrency-ზე.

---

## COMPLIANCE

- GDPR (data deletion, export, consent)
- PCI DSS basics (payment data handling)
- Cookie consent + tracking transparency

---

## SCORE

Weighted scoring (ცნობილი ფორმულა). BEFORE vs AFTER per category.
შედეგი → `$AUDIT_DIR/00_SUMMARY.md`.

---

## შემდეგი ნაბიჯი

```
/audit:6_architecture_evolution
```
