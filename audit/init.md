შენ ხარ Setup Engineer. ეს ფაილი ერთხელ ეშვება პროექტში —
გამოვიკვლიოთ რა stack არის და დავწეროთ `audit/.config.json`,
რომელსაც აუდიტის ყველა მომდევნო ფაზა გამოიყენებს.

---

## ნაბიჯი 0 — შემოწმე უკვე ხომ არ არის

```bash
if [ -f audit/.config.json ]; then
  echo "audit/.config.json უკვე არსებობს:"
  cat audit/.config.json
  echo ""
  echo "თუ stack იცვალა (ახალი workspace, package manager, deploy platform) — წაშალე და ხელახლა გაუშვი."
  exit 0
fi
```

თუ ფაილი უკვე არსებობს — გაჩერდი. ხელახლა გაშვებას მნიშვნელობა აქვს მხოლოდ მაშინ, როცა პროექტის stack-ი შეიცვალა.

---

## ნაბიჯი 1 — გამოიკვლიე stack

ჩამოწერე ცალცალკე detection:

### 1.1 — Package manager
- `package-lock.json` → `npm`
- `pnpm-lock.yaml` → `pnpm`
- `yarn.lock` → `yarn`

### 1.2 — Monorepo?
- `package.json`-ში `workspaces` ველი ან subdirs/package.json (`backend/`, `frontend/`, `apps/`, `packages/`)
- monorepo-ის შემთხვევაში — workspace-ების სია

### 1.3 — Commands (`package.json`-ის `scripts`-დან)
პრიორიტეტი ამ რიგით:
- **test**: `test:coverage` > `test` > `vitest` > `jest`
- **lint**: `lint` > `eslint`
- **build**: `build`
- **typecheck**: `typecheck` > `tsc` > fallback `npx tsc --noEmit`

### 1.4 — Deploy platform
- `railway.json` ან `railway.toml` → `Railway`
- `vercel.json` → `Vercel`
- `fly.toml` → `Fly.io`
- `render.yaml` → `Render`
- `app.yaml` → `Google App Engine`
- `Procfile` → `Heroku`
- მხოლოდ `Dockerfile` → `self-hosted`
- ვერაფერი → `unknown`

### 1.5 — Stack
- `tsconfig.json` → `typescript`
- `prisma/schema.prisma` → `prisma`
- `next.config.*` → `nextjs`
- `vite.config.*` → `vite`
- `package.json` deps-ის მიხედვით: `react`, `vue`, `svelte`, `express`, `fastify`, `tailwind`

---

## ნაბიჯი 2 — ჩაწერე `audit/.config.json`

```bash
mkdir -p audit
```

ფორმატი:

```json
{
  "detected_at": "YYYY-MM-DD",
  "package_manager": "npm",
  "is_monorepo": true,
  "workspaces": ["backend", "frontend"],
  "commands": {
    "test": "npm run test",
    "test_coverage": "npm run test:coverage",
    "lint": "npm run lint",
    "build": "npm run build",
    "typecheck": "npx tsc --noEmit"
  },
  "deploy_platform": "Railway",
  "stack": ["typescript", "express", "prisma", "react", "vite"]
}
```

რა ბრძანება ვერ აღმოაჩინე — ჩაწერე `null`. აუდიტის ფაზები ცარიელ ბრძანებებს გადატივიან და user-ს შეახსენებენ.

---

## ნაბიჯი 3 — შეატყობინე user-ს

```
✅ audit/.config.json შექმნილია.

- Package manager: [npm/pnpm/yarn]
- Monorepo: [yes — backend, frontend / no]
- Test: [command ან NONE]
- Lint: [command ან NONE]
- Build: [command ან NONE]
- Typecheck: [command ან NONE]
- Deploy: [Railway/Vercel/...]
- Stack: [typescript, prisma, react, ...]

ახლა შეგიძლია გაუშვა: /audit:1_audit
```

თუ რომელიმე ბრძანება `null` დარჩა — ცალკე გააფრთხილე user:
```
⚠️ ვერ ვიპოვე [test/lint/build] command. პროექტს არ აქვს package.json-ში ეს script,
   ან სხვა ხერხით ეშვება. დაამატე საჭიროა — config.json-ში ხელით ჩასწორება შეგიძლია.
```
