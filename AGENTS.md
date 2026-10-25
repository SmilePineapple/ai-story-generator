# AGENTS.md

Onboarding notes for AI coding agents. Last verified against the code on 2026-10-25 (commit `2921935`).

## What this is

`ai-story-generator` is meant to become a web app that generates stories with an LLM. **Right now it is only an
unmodified `create-next-app` scaffold** (Next.js App Router + Tailwind) with a few UI dependencies added. There is
no story-generation code yet: no prompts, no API routes, no LLM SDK, no env vars, no tests, no CI. Nothing in the
code says who the users are. Don't assume features exist because of the repo name.

## Tech stack

| Area | What | Version (package.json) |
|---|---|---|
| Framework | Next.js, App Router, `src/` dir | `15.0.3` (pinned) |
| UI runtime | React / React DOM | `19.0.0-rc-66855b96-20241106` (a **release candidate**, pinned) |
| Language | TypeScript, `strict: true` | `^5` |
| Styling | Tailwind CSS v3 + PostCSS | `^3.4.1` |
| UI deps (installed, **not yet imported anywhere**) | `@radix-ui/react-dialog`, `@radix-ui/react-icons`, `@radix-ui/react-slot`, `lucide-react` | `^1.1.2`, `^1.3.1`, `^1.1.0`, `^0.456.0` |
| Lint | ESLint 8, `next/core-web-vitals` + `next/typescript` | `^8`, `eslint-config-next 15.0.3` |
| Package manager | npm (`package-lock.json` committed) | tested on Node 22.22.0 / npm 10.9.4 |

Radix Slot + lucide + Tailwind usually means shadcn/ui-style components are planned, but there's no
`components.json`, `src/components/`, or `cn()`/`clsx` helper yet.

## Repo map

```
src/app/layout.tsx     Root layout; loads the Geist fonts via next/font/local; metadata is still "Create Next App"
src/app/page.tsx       Home page (/) — still the create-next-app boilerplate
src/app/globals.css    Tailwind directives + --background/--foreground CSS vars (dark mode via prefers-color-scheme)
src/app/fonts/         GeistVF.woff, GeistMonoVF.woff
public/                Boilerplate SVGs (next, vercel, file, window, globe)
next.config.ts         Empty config
tailwind.config.ts     content globs: src/pages, src/components, src/app; colors map to the CSS vars
tsconfig.json          strict; path alias @/* -> ./src/*
.eslintrc.json         next/core-web-vitals + next/typescript
README.md              Stock create-next-app README (says `app/page.tsx`; really it's `src/app/page.tsx`)
```

## Commands

Everything below was run in a cloud container on 2026-10-25 unless marked otherwise.

```bash
npm ci --legacy-peer-deps   # install. Plain `npm ci` / `npm install` FAILS (ERESOLVE, see Gotchas)
npm run dev                 # dev server with Turbopack on http://localhost:3000 (verified: returns 200)
npm run build               # production build (verified: passes; also runs lint + type check)
npm start                   # serve the production build (verified: returns 200 after build)
npm run lint                # next lint (verified: no warnings or errors)
npx tsc --noEmit            # type check on its own (verified: clean, but only after install)
```

- **Tests:** none exist. There is no test runner or `test` script.
- **CI:** none (no `.github/workflows`).
- **Deploy:** nothing is set up in the repo (no `vercel.json`, Dockerfile, or deploy script). The README points to
  Vercel, and `.gitignore` ignores `.vercel`, so Vercel is the likely target. That wasn't verified.
- No secrets are needed for any of these commands today.

## Architecture and data flow

Right now there is just one statically prerendered page (`/`), plus the default `/_not-found`. There is no backend,
no data layer, and no LLM call.

When you add story generation, these defaults fit the stack (they are suggestions, not existing code):
- Call the LLM on the server, either from a Route Handler (`src/app/api/<name>/route.ts`) or a Server Action, so API
  keys never reach the browser. Read keys from `process.env`, and never prefix secrets with `NEXT_PUBLIC_`.
- Put prompts and LLM-client code in their own module (e.g. `src/lib/`) instead of inline in components.
- Put shared UI in `src/components/`. It's already in the Tailwind `content` globs.
- Import with the `@/` alias, e.g. `import { x } from "@/lib/x"`.

## Environment variables

**None are used yet.** `.gitignore` ignores `.env*`, so `.env.local` is the place for local secrets. When you add
an env var, list it by name here and add a committed `.env.example` (you'll need a `!.env.example` negation in
`.gitignore`, because the `.env*` rule would otherwise ignore it).

## Conventions

- TypeScript everywhere, strict mode. Function components, default-exported for Next.js pages and layouts.
- Style with Tailwind utility classes. Theme colors come from CSS variables in `globals.css`.
- Double quotes and semicolons, as the generated code does. No Prettier config exists.
- Git: there's one commit (`Initial commit: AI Story Generator setup`, Nov 2024) on `main`. No branching or release
  process has been established yet. Agent work goes on `claude/<topic>` branches.

## Gotchas and known issues

1. **Peer-dependency conflict breaks install.** `@radix-ui/react-icons@1.3.1` declares a peer of `react@^19.x`,
   and npm doesn't count the pinned React RC (`19.0.0-rc-…`) as satisfying it. Use `--legacy-peer-deps`. The real
   fix is to upgrade to stable `react`/`react-dom` 19 (and `@types/react` 19). That is an app change, so ask
   the owner first.
2. **Type mismatch:** `@types/react` and `@types/react-dom` are `^18`, but the runtime is React 19 RC. That's fine
   for now, but React 19-only APIs may be missing types.
3. **Security:** `npm audit` reports 14 vulnerabilities (1 critical, 10 high, 3 moderate). The critical one is in
   `next@15.0.3` itself (e.g. GHSA-7m27-7ghc-44w9). Bump Next to a patched 15.x before deploying anything public.
4. `next lint` is deprecated in newer Next versions. Expect to migrate to the ESLint CLI / flat config when you
   upgrade.
5. `npm run dev` uses `--turbopack`. If you hit a dev-only bug, try `npx next dev` without it.
6. Running `npx tsc` before you install anything gives misleading JSX `any` errors. Install first.
7. Boilerplate still to replace: `metadata` in `layout.tsx`, the whole of `page.tsx`, the `public/*.svg` icons, and
   the README.
8. `globals.css` sets `font-family: Arial` on `body`, which overrides the Geist font variables unless a
   `font-[family-name:var(--font-geist-sans)]` class is used, as `page.tsx` does.

## Where to start

- **Build the story UI:** replace `src/app/page.tsx`. Add components under `src/components/`.
- **Add LLM generation:** create `src/app/api/generate/route.ts` (or a Server Action) and `src/lib/` for the client
  and prompts. Add the provider SDK with `npm install <sdk> --legacy-peer-deps`. Document the API-key env var name
  above.
- **Fix the install / security problems:** see Gotchas 1–3. Check with `npm ci`, `npm run build`, and `npm run lint`.
- **Add tests:** none exist, so you'll be choosing a runner (Vitest or Jest + Testing Library are the usual picks
  for Next). Add a `test` script and record the command here.
- **Before you finish any change:** run `npm run lint` and `npm run build`. Together they cover lint and types.
