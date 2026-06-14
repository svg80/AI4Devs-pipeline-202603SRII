Generate an OpenCode skill file named `devops-pipeline.md` inside `.opencode/skills/` that serves as an executable guide for an AI agent acting as a Senior DevOps Engineer on this project.

## Context

This is an educational full-stack project (React frontend, Express/Prisma backend, PostgreSQL). The CI pipeline at `.github/workflows/ci.yml` is currently **empty** — students build it from scratch. The target deployment is an AWS EC2 instance behind GitHub Actions.

Key project facts the skill must reference:
- Backend runs on port **3010**, frontend on **3000**
- Docker Compose starts PostgreSQL on port **5432** with env vars `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_PORT`
- Required GitHub Secrets: `AWS_ACCESS_ID`, `AWS_ACCESS_KEY`, `EC2_INSTANCE`
- Backend build: `npm run build` (tsc), dev: `npm run dev` (ts-node-dev), test: `npm test` (jest)
- Frontend build: `npm run build` (react-scripts), test: `npm test -- --watchAll=false`, E2E: `npm run cypress:run`
- `.env` is committed (not gitignored), but production credentials must use GitHub Secrets

## Skill content requirements

The skill must instruct an AI agent on how to:

1. **Design a CI/CD pipeline in `.github/workflows/ci.yml`** with four jobs: `lint`, `test`, `build`, `deploy` — each with a dependency on the previous.

2. **Lint job**: Run ESLint on the backend only (`cd backend && npx eslint src/`), fail on warnings.

3. **Test job**: Run backend Jest tests and frontend tests independently. Backend does not need a real DB for tests (tests are unit-level, mocking Prisma). Frontend tests run with `--watchAll=false`.

4. **Build job**: Compile backend (`npm run build`) and build frontend (`npm run build`). Cache `node_modules` and `backend/dist` / `frontend/build` where sensible.

5. **Deploy job**: Trigger only on push to `main`/`master`. Steps should include:
   - Check out code
   - Install system deps on EC2 via SSH (`rsync`, `pm2`, etc.)
   - Sync `backend/dist/` and `frontend/build/` to EC2 using `rsync` over SSH
   - SSH into EC2, copy the systemd/PM2 service file or restart the app via `pm2 restart`
   - Use the AWS Secrets stored in GitHub Secrets

6. **AWS best practices**:
   - Use `ssh` with a temporary SSH key written from a GitHub Secret (the key is stored as `SSH_PRIVATE_KEY`)
   - Security group for EC2 must allow ports 22, 80, 3010
   - Never hardcode IPs or credentials; always pull from `secrets.*` or `vars.*`

7. **GitHub Actions best practices**:
   - Use `actions/cache@v4` for dependency caching
   - Use `actions/checkout@v4`
   - Pin action versions to major versions
   - Fail-fast: `fail-fast: true` on matrix strategy if used
   - Add a `workflow_dispatch` trigger for manual runs
   - Namespace steps clearly with `name:` and use `|| true` sparingly

8. **Include the full YAML content** of the pipeline inline in the skill as a ready-to-copy example.

9. **Security notes**: warn the agent never to log secrets, avoid `echo $SECRET`, avoid exposing private key contents in workflow output.

10. **Verify the pipeline**: instructions to validate the workflow using `act` (nektos/act) locally if available, or by pushing to a feature branch and checking Action runs.

## Output format

Write the skill as a valid OpenCode skill file with:
- `## Skill: DevOps Pipeline — GitHub Actions + AWS EC2` as the title
- A `<context>` section describing when this skill applies
- A `<instructions>` section with the numbered steps above translated into actionable commands the agent should run
- A `<examples>` section with the full `.github/workflows/ci.yml` YAML
- A `<verification>` section with commands to run locally to verify correctness

## Constraints

- Do NOT write any actual code, config, or pipeline YAML as part of **this prompt**. The prompt must only instruct *how* and *what* to generate.
- All project paths are relative to the repository root.
- Assume Ubuntu 22.04 for the runner and Amazon Linux 2 for the EC2 target.
- The EC2 instance already has Node.js v18+, PM2, and Nginx installed.
