# AGENTS.md — LTI Talent Tracking System

## Project structure

- **`backend/`** — Express + TypeScript + Prisma (PostgreSQL). Entry: `src/index.ts`, port 3010.
- **`frontend/`** — Create React App (TypeScript). Entry: `src/index.tsx`, port 3000, uses react-router-dom.
- **`/`** root `package.json` only holds `dotenv` dep + Prisma schema pointer (`"prisma.schema": "backend/prisma/schema.prisma"`).

## Commands

```sh
# Backend (run from backend/)
npm run dev        # ts-node-dev with hot reload
npm run build      # tsc -> dist/
npm start          # node dist/index.js
npm test           # jest (ts-jest, node env)

# Frontend (run from frontend/)
npm start          # react-scripts dev server
npm run build      # react-scripts build -> build/
npm test           # jest --config jest.config.js
npm run cypress:open  # Cypress interactive
npm run cypress:run   # Cypress headless
```

## Database setup

```sh
docker-compose up -d                        # start PostgreSQL
cd backend
npx prisma generate && npx prisma migrate dev && ts-node prisma/seed.ts
```

- `.env` is **not** gitignored (`.gitignore` has `#**/.env`). Prod credentials must use GitHub Secrets.
- `DATABASE_URL` env var expected in `backend/.env`. See `.env` at repo root for defaults.

## Architecture

DDD-inspired layers in `backend/src/`:
- **domain/models/** — entity classes with `save()`, `findOne()`, etc. Mirror Prisma models.
- **application/services/** — business logic (candidateService, positionService).
- **presentation/controllers/** — Express request handlers. Thin wrappers around services.
- **routes/** — Express Router wiring.

Reference: `backend/ManifestoBuenasPracticas.md` (DDD guide, 1215 lines).
API spec: `backend/api-spec.yaml`. Data model: `backend/ModeloDatos.md`.

Frontend routes (react-router-dom):
- `/` — RecruiterDashboard
- `/add-candidate` — AddCandidate
- `/positions` — Positions list
- `/positions/:id` — PositionDetails

## Testing

- Backend tests use Jest + ts-jest. 4 test files exist:
  - `backend/src/presentation/controllers/candidateController.test.ts`
  - `backend/src/presentation/controllers/positionController.test.ts`
  - `backend/src/application/services/candidateService.test.ts`
  - `backend/src/application/services/positionService.test.ts`
- Frontend has **no unit tests**; Cypress E2E tests in `frontend/cypress/`.

## Code style

- **Prettier**: single quotes, trailing commas everywhere (`backend/.prettierrc`).
- **ESLint**: extends `plugin:prettier/recommended` (`backend/.eslintrc.js`).
- **TypeScript**: v4.9.5 in both packages. Backend targets ES5/commonjs; frontend targets ES5 with JSX.

## CI/CD

`.github/workflows/ci.yml` is **empty** — this is an educational project where students build the pipeline. AWS secrets (`AWS_ACCESS_ID`, `AWS_ACCESS_KEY`, `EC2_INSTANCE`) must be set as GitHub Secrets.

## AI-assisted dev

- `backend/src/prompts/CreateNewRoute.md` — template prompt for creating endpoints.
- `prompts/prompts.md` and `prompts/prompts-svg.md` — empty prompt files for student use.
