# Freelancer Website

Strona osobista freelancera-programisty (marka osobista) — Next.js 14 App Router + Headless WordPress (WPGraphQL + ACF) + Stripe. Monorepo pnpm.

## Status projektu

🚧 W budowie. Implementacja według planu w ARCHITEKTURA.md (sekcja 8 — 42 zadania).
Ukończone: TASK-01 (monorepo), TASK-02 (konfiguracja).

## Wymagania

- Node.js 20+ (patrz `.nvmrc`)
- pnpm 9.12+ (Corepack: `corepack enable`)
- Docker + Docker Compose (lokalny WordPress)
- Git

## Struktura monorepo

Główne katalogi monorepo:
- `apps/web/` — Aplikacja frontendowa Next.js 14+ (App Router, React Server Components, Tailwind CSS, Shadcn UI, Framer Motion)
- `packages/shared/` — Pakiet ze współdzielonymi schematami Zod, typami TypeScript oraz zapytaniami/mutacjami GraphQL
- `wp-plugin/` — Dedykowany plugin PHP (`freelancer-cms-core`) oraz środowisko Docker dla Headless WordPress

## Setup lokalny

1. Klonowanie i instalacja:
```bash
git clone https://github.com/krzysztof86/freelancer-website.git
cd freelancer-website
nvm use
corepack enable
pnpm install
```

2. Zmienne środowiskowe:
```bash
cp .env.example .env.local
# uzupełnij klucze API w .env.local
```

3. WordPress (wymaga TASK-03):
```bash
cd wp-plugin
docker compose up -d
# Panel WP: http://localhost:8000/wp-admin
# GraphQL: http://localhost:8000/graphql
```

4. Generowanie typów GraphQL (wymaga TASK-07 i TASK-11):
```bash
pnpm codegen
```

5. Uruchomienie dev serwera:
```bash
pnpm dev
# Next.js: http://localhost:3000
```

## Skrypty

| Skrypt | Opis |
| :--- | :--- |
| `pnpm dev` | Uruchamia serwer deweloperski Next.js dla aplikacji web (placeholder — implementacja w TASK-10) |
| `pnpm build` | Buduje produkcyjne wersje aplikacji i pakietów (placeholder — implementacja w TASK-10) |
| `pnpm test` | Uruchamia zestaw testów jednostkowych Vitest (placeholder — implementacja w TASK-12) |
| `pnpm codegen` | Generuje typowane hooks i typy TypeScript z WPGraphQL (placeholder — implementacja w TASK-11) |
| `pnpm lint` | Przeprowadza statyczną analizę kodu ESLint / Prettier (placeholder — implementacja w TASK-10) |

## Dokumentacja

- `ARCHITEKTURA.md` — architektura, model danych CPT, plan 42 zadań
- `.env.example` — lista wymaganych zmiennych środowiskowych
- `wp-plugin/` — plugin PHP (rejestracja CPT, ACF, mutacje GraphQL)

## Deployment

Dwa środowiska: Vercel (frontend Next.js) + osobny hosting WordPress (DEC-01 w ARCHITEKTURA.md). Pełna konfiguracja w TASK-42.

## Troubleshooting

### `pnpm install` fails on packageManager
Upewnij się, że Corepack jest aktywny: `corepack enable`

### Port 8000 zajęty (WordPress)
Sprawdź: `lsof -i :8000` i zatrzymaj proces lub zmień port w `docker-compose.yml`

### GraphQL 401 Unauthorized
Sprawdź czy `WP_APPLICATION_PASSWORD_USER` i `WP_APPLICATION_PASSWORD` są ustawione w `.env.local` (TASK-07)

### TypeScript errors after pnpm install
Uruchom `pnpm codegen` — typy GraphQL muszą być wygenerowane przed `pnpm build`
