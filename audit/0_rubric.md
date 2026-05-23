აუდიტის ქულების მინიჭების კონკრეტული წესები. გამოიყენე `1_audit`-ში საწყისი ქულისთვის და `3b_verify_*`-ში re-score-ისთვის. ეს ფაილი არ ცვლის სხვა ფაზებს — მხოლოდ მათ ანდევს გადაწყვეტილების ერთიან წესს.

---

## საწყისი ქულის შკალა (`1_audit`)

თითოეული კატეგორია — 0-დან 10-მდე:

| ქულა | ნიშნავს | მაგალითი |
|------|---------|----------|
| 9-10 | წარჩინებული | best practice, არანაირი issue, evidence-based |
| 7-8  | კარგი | მცირე გასაუმჯობესებელი, არანაირი blocker |
| 5-6  | საშუალო | რეალური problems, არა critical |
| 3-4  | ცუდი | bones broken, ბევრი fix-ი სჭირდება |
| 0-2  | კატასტროფული | production-ისთვის გადასააქცევი |

---

## Re-score Delta წესები (`3b_verify_*`)

ცვლილებები **evidence-based**, არა subjective. შეადარე BEFORE → AFTER:

### დადებითი ცვლილებები (+)

| შემთხვევა | Delta |
|-----------|-------|
| 🔴 Critical issue გადაჭრილი + regression test დაიწერა | **+1.0** |
| 🔴 Critical issue გადაჭრილი, ტესტი არ | **+0.5** |
| 🟡 High issue გადაჭრილი | **+0.5** |
| 🟢 Medium issue გადაჭრილი | **+0.2** |
| Test coverage +10% (მაგ. 60% → 70%) | **+0.5** |
| Test coverage +20% | **+1.0** |
| Lint errors 0-მდე ჩამოვიდა (იყო > 0) | **+0.5** |
| TypeScript `any` count −50% | **+0.5** |
| Files > 400 lines: count −50% | **+0.3** |
| ახალი security headers / CSP / helmet | **+0.5** |
| New documentation (README sections, .env.example up-to-date) | **+0.3** |
| Circular dependency გადაჭრილი | **+0.3** |

### უარყოფითი ცვლილებები (−)

| შემთხვევა | Delta |
|-----------|-------|
| Regression — იყო passing test, ახლა failing | **−1.5** |
| Test coverage −10% | **−1.0** |
| New lint errors | **−0.3 per error** |
| New TypeScript `any` | **−0.2 per occurrence** |
| New circular dependency | **−0.5** |
| Build broken | **−2.0** (instant STOP — verify ფაზა აჩერდეს) |
| Security regression (header მოშორდა, validation მოშორდა) | **−1.0** |

### Ceiling / Floor

- კატეგორიის ქულა ვერ აჯობებს **10**-ს
- ვერ ჩამოვა **0**-ს ქვემოთ
- თუ კატეგორიის ყველა issue გადაჭრილია + ტესტები passing + lint clean → ქულა იწევს მინიმუმ **9.0**-მდე
- 🔴 Critical issues outstanding → კატეგორიის maximum **6.0** (ცეცხლი ვერ ჩაცხრილდი)

---

## Overall Score ფორმულა

```
overall = (security × 0.20) + (code_quality × 0.15) + (testing × 0.15)
        + (architecture × 0.10) + (error_handling × 0.10) + (frontend × 0.10)
        + (cicd × 0.05) + (database × 0.05) + (git × 0.05) + (documentation × 0.05)
```

**Frontend არ არსებობს** → security 25%, testing 20%, frontend 0%, დანარჩენი იგივე.
სხვა კატეგორიის გადანაწილება (მაგ. backend არ არსებობს, frontend-only) — დოკუმენტირდეს `00_DECISIONS_LOG.md`-ში.

---

## Honest Scoring წესი

ქულა აიწიოს **მხოლოდ რეალური improvement-ის შემთხვევაში**:
- კოდი მთლიანად შეიცვალა, მაგრამ ფუნქცია იგივეა → **ქულა არ იცვლება**
- ტესტი დაემატა, მაგრამ assertion-ი არ აქვს → **ქულა არ იცვლება**
- Refactor-ი kosmetikuri (ცვლადის სახელი) → **ქულა არ იცვლება**
- რეგრესია გამოვლინდა → **ქულა იწევს, fix-ის მიუხედავად**

თუ verify ფაზის ბოლოს ცხრილში `Before` და `After` ემთხვევა — ეს ნორმაა, არა შეცდომა. ნუ აიწევ ხელოვნურად.

---

## Verify Output ცხრილი

3b_verify_* ფაზებში ბოლოს:

```markdown
| Category       | Start | After Critical | After High | After Medium | Final |
|----------------|-------|----------------|------------|--------------|-------|
| Security       | 4.5   | 7.0            | 7.5        | 7.5          | 7.5   |
| Code Quality   | 6.0   | 6.0            | 6.5        | 8.0          | 8.0   |
| Testing        | 3.0   | 3.5            | 6.5        | 6.5          | 6.5   |
| ...            | ...   | ...            | ...        | ...          | ...   |
| **Overall**    | 4.8   | 5.6            | 6.5        | 7.0          | 7.0   |
```

თითოეული 3b ფაზა **ცალკე სვეტს** ამატებს — წინა სვეტებს არ ცვლის.
ეს history ბოლოს `7_final`-ის HTML dashboard-ში გადადის როგორც evidence.
