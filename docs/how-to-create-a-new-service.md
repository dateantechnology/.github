# How to Create a New Service

This guide walks you through setting up a new repository in the **dateantechnology** org
from scratch — correctly, from day one. Follow every step; skipping steps causes
CI/CD problems later.

**Prerequisite:** You must be a member of the **platform-leads** team, or work with one,
to complete steps 4–7 (they require admin-level repo permissions).

---

## Naming Conventions

Repo names must be:
- **kebab-case**, all lowercase
- **Prefixed with the team** that owns the service
- **Descriptive of function**, not technology

| Pattern | Example | Meaning |
|---------|---------|---------|
| `<team>-www` | `act-www` | Public-facing website for ACT |
| `<team>-admin` | `cog-admin` | Admin panel for COG-IT |
| `<team>-www-bff` | `cog-www-bff` | BFF Lambda for COG's website |
| `<team>-api` | `rdit-api` | Internal API for R&DIT |
| `<team>-ingestor` | `rdit-ingestor` | Data ingestor for R&DIT |
| `<team>-app` | `act-app` | Mobile app for ACT |

**Avoid:** `new-service`, `backend`, `frontend`, `test-project`, `my-lambda`

---

## Step 1 — Pick the Right Template Repository

Choose the template that matches your stack:

| Stack | Template repo |
|-------|---------------|
| Next.js + TypeScript website | `dateantechnology/template-nextjs` |
| Next.js + TypeScript admin panel | `dateantechnology/template-nextjs-admin` |
| Python AWS SAM Lambda (BFF / API) | `dateantechnology/template-python-sam` |
| Python AWS SAM Lambda (ingestor) | `dateantechnology/template-python-sam-ingestor` |
| Flutter mobile app | `dateantechnology/template-flutter` |

---

## Step 2 — Create the Repository from Template

1. Navigate to the chosen template repo on GitHub.
2. Click the green **"Use this template"** button → **"Create a new repository"**.
3. Set:
   - **Owner:** `dateantechnology`
   - **Repository name:** following the [naming convention](#naming-conventions) above
   - **Visibility:** Private (always — unless explicitly agreed otherwise)
   - **Description:** One sentence describing what this service does
4. Click **"Create repository"**.

> Do **not** tick "Include all branches" — the template's default branch is all you need.

---

## Step 3 — Configure Repository Settings

Navigate to **Settings** in the new repo and apply the following:

### General

| Setting | Value |
|---------|-------|
| Default branch | `main` |
| Merge strategy | ✅ Allow squash merging only (disable merge commits and rebase) |
| Auto-delete head branches | ✅ Enabled |
| Allow auto-merge | ✅ Enabled |

### Branch protection — `main`

Go to **Settings → Branches → Add rule**:

| Rule | Value |
|------|-------|
| Branch name pattern | `main` |
| Require a pull request before merging | ✅ |
| Required approvals | 1 |
| Dismiss stale reviews on new commits | ✅ |
| Require status checks to pass | ✅ (add CI job names after first run) |
| Require branches to be up to date | ✅ |
| Require signed commits | ✅ |
| Include administrators | ✅ |

### Branch protection — `env/test` and `env/prod`

Apply the same rule to both environment branches, but adjust:
- Required approvals: 0 for `env/test`, 1 for `env/prod`
- These branches receive squash-merges from `main` only

---

## Step 4 — Create the Environment Branches

```bash
# Clone the new repo
gh repo clone dateantechnology/<repo-name>
cd <repo-name>

# Create test and prod branches
git checkout -b env/test
git push origin env/test

git checkout main
git checkout -b env/prod
git push origin env/prod
```

---

## Step 5 — Set Up GitHub Environments

Go to **Settings → Environments** and create two environments:

### `test` environment

| Setting | Value |
|---------|-------|
| Name | `test` |
| Deployment branch rule | Selected branches → `env/test` |
| Required reviewers | None |
| Wait timer | None |

### `production` environment

| Setting | Value |
|---------|-------|
| Name | `production` |
| Deployment branch rule | Selected branches → `env/prod` |
| Required reviewers | 1 reviewer from `platform-leads` |
| Wait timer | Optional (5 minutes recommended) |

---

## Step 6 — Add Required Repository Variables and Secrets

Go to **Settings → Secrets and variables → Actions**.

### Variables (non-secret, per environment)

Add these under each environment:

| Variable | `test` example | `production` example |
|----------|---------------|---------------------|
| `AWS_REGION` | `eu-west-1` | `eu-west-1` |
| `S3_BUCKET_NAME` | `act-www-test-assets` | `act-www-prod-assets` |
| `CLOUDFRONT_DISTRIBUTION_ID` | `E1ABCDEF1234` | `E9ZYXWVU5678` |

### Repository-level variables (non-secret)

| Variable | Value |
|----------|-------|
| `AWS_ACCOUNT_ID` | Your AWS account ID |

### Repository-level secrets (sensitive)

| Secret | Description |
|--------|-------------|
| `AWS_OIDC_ROLE_ARN` | ARN of the GitHub Actions OIDC IAM role (get from platform-leads) |

> **Never** store AWS access keys as secrets. All AWS authentication uses OIDC.
> The OIDC role ARN follows the pattern:
> `arn:aws:iam::<account-id>:role/github-actions-<repo-name>`

---

## Step 7 — Update CODEOWNERS

Open `.github/CODEOWNERS` in the new repo and assign ownership:

```
# .github/CODEOWNERS
# Syntax: <path pattern>  <@owner>

# Default — the owning team reviews everything
*  @dateantechnology/<team-name>

# CI/CD config — platform-leads must review
.github/  @dateantechnology/platform-leads

# Infrastructure config (SAM template, CDK, etc.)
template.yaml  @dateantechnology/platform-leads
```

Replace `<team-name>` with `act`, `cog-it`, or `rdit` as appropriate.

Commit this to `main`:
```bash
git add .github/CODEOWNERS
git commit -m "chore: add CODEOWNERS"
git push origin main
```

---

## Step 8 — Add to the Engineering Backlog Project

1. Go to the **dateantechnology** GitHub org → **Projects**.
2. Open the **Engineering Backlog** project.
3. Click **+ Add item** and link the new repository.
4. Create an initial ticket: `"DT-XXX: Initial setup and first deployment for <repo-name>"`.
5. Assign it to yourself and set status to **In Progress**.

---

## Step 9 — Run the First CI Check

Push a small change to trigger the CI workflow and confirm everything is wired correctly:

```bash
# Make a trivial change
echo "# $(date)" >> .github/.first-run
git add .
git commit -m "chore: trigger initial CI run"

# Open a PR
git checkout -b chore/initial-ci-check
git push origin chore/initial-ci-check
gh pr create --base main --title "chore: initial CI check" \
  --body "Verifying CI pipeline is fully configured."
```

Watch the **Actions** tab. Once the CI run passes:
1. Merge the PR (squash merge).
2. Confirm the status checks are visible on the `main` branch protection rule
   and add them if they aren't yet.
3. Post in your team channel that the new service is ready.

---

## Post-setup Checklist

- [ ] Repo created from correct template
- [ ] Repo name follows naming convention
- [ ] Squash-only merge strategy enabled
- [ ] Auto-delete branches enabled
- [ ] Branch protection on `main` (PR + 1 approval + signed commits)
- [ ] `env/test` and `env/prod` branches created
- [ ] `test` and `production` GitHub Environments configured
- [ ] `AWS_OIDC_ROLE_ARN` secret added
- [ ] S3 / CloudFront variables added per environment
- [ ] CODEOWNERS updated
- [ ] Added to Engineering Backlog project
- [ ] First CI run passed
```

---
