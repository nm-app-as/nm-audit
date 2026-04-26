"Would you invest in this codebase?" test.
Due diligence team — acquisition-ისთვის რომ აფასებდე.

---

## ფაზა 0 — ყველა Evidence

```bash
AUDIT_DIR="audit/$(cat audit/LATEST)"

# არქიტექტურული გადაწყვეტილებები
cat docs/decisions/*.md 2>/dev/null

# Fix history
cat "$AUDIT_DIR/00_PROGRESS_TRACKER.md"
cat "$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md"
cat "$AUDIT_DIR/00_SUMMARY.md"

# Git history
git log --oneline -50

# იგივე branch
git branch --show-current
```

Due diligence team-მა ყველაფერი წაიკითხა სანამ აფასებდა. შენც გააკეთე.

---

## 8 კითხვა — evidence-ით

1. **Maintainability**: 3-5 კაციანი გუნდი maintain-ს გაუძლებს?
   - Evidence: onboarding time (audit:6-დან), test coverage %, README სრულობა

2. **Technical debt**: დევ-საათებში რა შეფასდება?
   - Evidence: კონკრეტული ფაილები, line counts, TODO comments, open issues

3. **Bus factor**: რამდენი კაცი თუ წავა — პროექტი მოკვდება?
   - Evidence: `git log --format='%an' | sort | uniq -c | sort -rn` — unique authors per critical file

4. **Top 5 risks**: რომელი 5 რამე გამოიწვევს production incident-ს?
   - Evidence: კონკრეტული endpoints, DB queries, infra gaps

5. **Downtime cost**: საათში რა ღირს business-ისთვის?
   - Evidence: თუ CLAUDE.md-ში არ წერია → UNKNOWN, არ მოიგონო

6. **Test confidence**: refactor-ისთვის საკმარისია?
   - Evidence: გაუშვი test suite, რეალური coverage რიცხვი

7. **Legal/compliance**: GDPR/PCI რისკები?
   - Evidence: 5_attack-ის findings

8. **Competitor exploit**: რას გამოიყენებდა მტერი პირველად?
   - Evidence: უმაღლესი CVSS finding 5_attack-დან

---

## FINAL REPORT FORMAT

```markdown
## Due Diligence Report — [date]

### Scores (ყველა წინა ფაზიდან)
| Category     | Start | Now  | Delta |
|--------------|-------|------|-------|
| Security     | XX%   | XX%  | +/-X  |
| ...          | ...   | ...  | ...   |

### Answers to 8 Questions
[თითოეული პასუხი evidence-ით]

### Investment Verdict
ერთი:
✅ GO — production-ready, რისკები known + acceptable
⚠️ CONDITIONAL GO — ready ამ კონკრეტული პირობებით: [list]
❌ NO-GO — ეს blockers ჯერ უნდა გადაწყდეს: [list]
```

verdict უნდა იყოს ამ სამიდან ერთი. ბუნდოვანი დასკვნა არა.

---

## ბოლო ფაზა — Visual HTML Dashboard

გენერირება საბოლოო HTML dashboard-ის — visual evidence.

```
$AUDIT_DIR/report/index.html   ← single self-contained, build tools-ის გარეშე
```

Browser-ში double-click-ით უნდა იხსნებოდეს. Chart.js CDN-დან:
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
```

### Data Sources
წაიკითხე და embed-ი ცვლადებად HTML-ში:
- `$AUDIT_DIR/00_SUMMARY.md` → overall summary, scores
- `$AUDIT_DIR/00_BEFORE_AFTER_METRICS.md` → metrics
- `$AUDIT_DIR/00_PROGRESS_TRACKER.md` → issues with status
- `$AUDIT_DIR/00_DECISIONS_LOG.md` → architectural decisions
- Final verdict (GO / CONDITIONAL GO / NO-GO)

### Sections (ამ რიგით)

1. **HEADER** — project name, date, overall score (large), verdict badge (✅ green / ⚠️ yellow / ❌ red)
2. **SCORE PROGRESSION (line chart)** — X: ფაზები, Y: 0–10. ერთი line per category.
3. **CURRENT STATE (radar chart)** — ყველა კატეგორია. "start" vs "now" — 2 polygon overlapping.
4. **BEFORE/AFTER METRICS (table)** — color-coded: green improved, red regressed, grey unchanged.
5. **ISSUES BY SEVERITY (horizontal bar)** — 🔴/🟡/🟢, stacked: resolved vs outstanding.
6. **TOP 5 RISKS (cards)** — Q4-ის პასუხები. severity, რომელი ფაზამ აღმოაჩინა.
7. **DECISIONS LOG (collapsible)** — `00_DECISIONS_LOG.md`-დან.

### Visual Style
- Dark background `#0f172a`, white text
- Accent: `#6366f1` (indigo)
- Green `#22c55e`, Yellow `#eab308`, Red `#ef4444`
- Plain CSS, no Bootstrap/Tailwind
- Mobile-friendly (CSS grid, responsive)

---

## CHANGELOG განახლება

```bash
# საბოლოო verdict + overall — დააფიქსირე CHANGELOG-ში
# (1_audit-მა დაწერა "(in progress)" ხაზი — ჩაანაცვლე საბოლოო-ით)
AUDIT_DATE=$(cat audit/LATEST)
sed -i.bak "s|^| $AUDIT_DATE | X.X | (in progress) ||| $AUDIT_DATE | [final-overall] | [VERDICT] |" audit/CHANGELOG.md
```

(ან ხელით განაახლე ბოლო ხაზი final overall + verdict-ით.)

---

## ბოლო შეტყობინება + PR offer

ბოლოს მოახსენე user-ს და სამი ვარიანტი შესთავაზე:

```
✅ აუდიტი დასრულებულია.

რეპორტი: $AUDIT_DIR/report/index.html — გახსენი browser-ში
Branch: audit/$(cat audit/LATEST) — ყველა ფიქსი აქ არის
Verdict: [✅ GO / ⚠️ CONDITIONAL GO / ❌ NO-GO]

შემდეგი ნაბიჯი — main-ში როგორ შევიტანოთ?

  (a) ცალკე გადახედე ფიქსები და ხელით merge-ე
  (b) გავაკეთო GitHub PR ახლავე (`gh pr create`)
  (c) abort — main-ს ვუბრუნდე, branch რჩება ხელუხლებელი (`/audit:abort`)

რომელი?
```

დაელოდე user-ის პასუხს. არავითარი ავტომატური merge ან PR — გადაწყვეტილება user-ზე.
