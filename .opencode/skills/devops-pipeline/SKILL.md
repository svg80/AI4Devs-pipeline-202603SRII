---
name: devops-pipeline
description: Build or improve a CI/CD pipeline with GitHub Actions and AWS EC2 deployment for a full-stack React/Express/Prisma app
---

## Skill: DevOps Pipeline — GitHub Actions + AWS EC2

<context>
This skill applies when the user asks you to build or improve a CI/CD pipeline for this project. The target is a GitHub Actions workflow that deploys to an AWS EC2 instance. The project is an educational full-stack app (React frontend on port 3000, Express/Prisma backend on port 3010, PostgreSQL via Docker Compose).

The workflow file at `.github/workflows/ci.yml` is currently empty — you need to create it from scratch.
</context>

<instructions>
You are a Senior DevOps Engineer. Create or update `.github/workflows/ci.yml` with four sequential jobs: `lint` -> `test` -> `build` -> `deploy`. Use `needs:` to chain them. The deploy job must run only on push to `main`/`master`.

### 1. Lint job
- Run ESLint on the backend only.
- Command: `cd backend && npx eslint src/`
- Fail the job if there are any warnings or errors.
- Runner: `ubuntu-latest`.

### 2. Test job
- Needs: `lint`
- Run backend tests and frontend tests in parallel steps within the same job.
- Backend: `cd backend && npm test`
- Frontend: `cd frontend && npm test -- --watchAll=false`
- Cache `node_modules` for both packages.
- Backend tests are unit-level (mock Prisma), so no database is needed.

### 3. Build job
- Needs: `test`
- Build both packages:
  - `cd backend && npm run build` (tsc -> `backend/dist/`)
  - `cd frontend && npm run build` (react-scripts -> `frontend/build/`)
- Cache `backend/dist/` and `frontend/build/` between runs to speed up the deploy step.

### 4. Deploy job
- Needs: `build`
- Trigger: only on `push` to `main` or `master` (use `if: github.ref == 'refs/heads/main'`).
- Steps:
  1. Check out code using `actions/checkout@v4`.
  2. Write the SSH private key from secrets: `echo "${{ secrets.SSH_PRIVATE_KEY }}" > /tmp/ssh_key && chmod 600 /tmp/ssh_key`.
  3. Install `rsync` on the runner: `sudo apt-get update && sudo apt-get install -y rsync`.
  4. Sync `backend/dist/` to EC2: `rsync -avz --delete -e "ssh -i /tmp/ssh_key -o StrictHostKeyChecking=no" backend/dist/ ubuntu@${{ secrets.EC2_INSTANCE }}:~/app/backend/dist/`.
  5. Sync `frontend/build/` to EC2 similarly to `~/app/frontend/build/`.
  6. Restart the app: `ssh -i /tmp/ssh_key ubuntu@${{ secrets.EC2_INSTANCE }} "pm2 restart all"`.
  7. Clean up the SSH key: `rm -f /tmp/ssh_key`.

### GitHub Secrets required
Set these in the repository Settings > Secrets and variables > Actions:
- `SSH_PRIVATE_KEY` — the private key for EC2 SSH access
- `EC2_INSTANCE` — public IP or DNS of the EC2 instance
- `AWS_ACCESS_ID` — AWS access key ID (if needed for other integrations)
- `AWS_ACCESS_KEY` — AWS secret access key

### Security rules (enforce these)
- Never `echo` a secret value or write it to a log file.
- Always `chmod 600` SSH keys before use.
- Always delete SSH keys from the runner filesystem after use.
- Use `secrets.*` and `vars.*` — never hardcode IPs, keys, or passwords.
- Pin action versions (`@v4`, never `@main` or `@latest`).
</instructions>

<examples>

### Full `.github/workflows/ci.yml`

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, master]
  workflow_dispatch:

env:
  NODE_VERSION: '18'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - name: Cache backend node_modules
        uses: actions/cache@v4
        with:
          path: backend/node_modules
          key: ${{ runner.os }}-backend-node-${{ hashFiles('backend/package-lock.json') }}
      - name: Cache frontend node_modules
        uses: actions/cache@v4
        with:
          path: frontend/node_modules
          key: ${{ runner.os }}-frontend-node-${{ hashFiles('frontend/package-lock.json') }}
      - name: Install backend deps
        run: cd backend && npm ci
      - name: Install frontend deps
        run: cd frontend && npm ci
      - name: Lint backend
        run: cd backend && npx eslint src/

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - name: Restore backend node_modules
        uses: actions/cache@v4
        with:
          path: backend/node_modules
          key: ${{ runner.os }}-backend-node-${{ hashFiles('backend/package-lock.json') }}
      - name: Restore frontend node_modules
        uses: actions/cache@v4
        with:
          path: frontend/node_modules
          key: ${{ runner.os }}-frontend-node-${{ hashFiles('frontend/package-lock.json') }}
      - name: Install backend deps
        run: cd backend && npm ci
      - name: Install frontend deps
        run: cd frontend && npm ci
      - name: Backend tests
        run: cd backend && npm test
      - name: Frontend tests
        run: cd frontend && npm test -- --watchAll=false

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      - name: Restore backend node_modules
        uses: actions/cache@v4
        with:
          path: backend/node_modules
          key: ${{ runner.os }}-backend-node-${{ hashFiles('backend/package-lock.json') }}
      - name: Restore frontend node_modules
        uses: actions/cache@v4
        with:
          path: frontend/node_modules
          key: ${{ runner.os }}-frontend-node-${{ hashFiles('frontend/package-lock.json') }}
      - name: Install deps
        run: |
          cd backend && npm ci
          cd ../frontend && npm ci
      - name: Build backend
        run: cd backend && npm run build
      - name: Build frontend
        run: cd frontend && npm run build
      - name: Cache build artifacts
        uses: actions/cache@v4
        with:
          path: |
            backend/dist
            frontend/build
          key: ${{ runner.os }}-build-${{ github.sha }}

  deploy:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Restore build artifacts
        uses: actions/cache@v4
        with:
          path: |
            backend/dist
            frontend/build
          key: ${{ runner.os }}-build-${{ github.sha }}
      - name: Setup SSH key
        run: |
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > /tmp/ssh_key
          chmod 600 /tmp/ssh_key
      - name: Install rsync
        run: sudo apt-get update && sudo apt-get install -y rsync
      - name: Sync backend build to EC2
        run: |
          rsync -avz --delete \
            -e "ssh -i /tmp/ssh_key -o StrictHostKeyChecking=no" \
            backend/dist/ \
            ubuntu@${{ secrets.EC2_INSTANCE }}:~/app/backend/dist/
      - name: Sync frontend build to EC2
        run: |
          rsync -avz --delete \
            -e "ssh -i /tmp/ssh_key -o StrictHostKeyChecking=no" \
            frontend/build/ \
            ubuntu@${{ secrets.EC2_INSTANCE }}:~/app/frontend/build/
      - name: Restart application
        run: |
          ssh -i /tmp/ssh_key -o StrictHostKeyChecking=no \
            ubuntu@${{ secrets.EC2_INSTANCE }} "pm2 restart all"
      - name: Cleanup SSH key
        if: always()
        run: rm -f /tmp/ssh_key
```

</examples>

<verification>
After creating the workflow, verify it with one of these methods:

**Option A — Local validation with `act`**
```sh
# Install act (one time)
curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Run lint job locally (requires Docker)
act -j lint

# Run full pipeline
act --workflows .github/workflows/ci.yml
```

**Option B — Push to a feature branch**
1. Commit the workflow file and push to a feature branch.
2. Go to https://github.com/YOUR_USER/YOUR_REPO/actions and confirm the workflow runs.
3. Check each job's output for errors.

**Option C — Manual workflow_dispatch**
After merging to main, trigger the workflow manually from the Actions tab using the `workflow_dispatch` button and verify deploy succeeds.

### Acceptance criteria
- [ ] `lint` job passes with no ESLint warnings
- [ ] `test` job runs all tests green
- [ ] `build` produces `backend/dist/` and `frontend/build/`
- [ ] `deploy` job connects to EC2 and restarts the app
- [ ] SSH key is cleaned up even if deploy fails
- [ ] No secrets appear in workflow logs
</verification>
