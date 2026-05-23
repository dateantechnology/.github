# Troubleshooting Guide

Common problems across all three stacks — and how to fix them.
Organised by where you are when things break.

Jump to a section:

- [CI Failures](#ci-failures)
- [Deployment Issues](#deployment-issues)
- [Local Dev Issues](#local-dev-issues)
- [GitHub Issues](#github-issues)

---

## CI Failures

### Lint errors

**Problem:** CI fails on the lint step with ESLint or ruff errors.

**Cause:** Code was pushed without running lint locally.

**Solution:**

```bash
# Next.js
npm run lint
npm run lint -- --fix   # auto-fix safe issues

# Python
source .venv/bin/activate
ruff check . --fix
ruff format .
mypy .
```

Fix remaining issues manually, commit, and push again.

---

### Type errors (TypeScript)

**Problem:** CI fails with `Type error: ...` in the `typecheck` step.

**Cause:** TypeScript type mismatch — often from a changed API response shape or
missing type annotation.

**Solution:**

```bash
npm run typecheck
# Read the error — it includes the exact file and line number

# Common fixes:
# 1. Missing return type: add explicit return type to function
# 2. `undefined` not handled: add null check or use optional chaining (?.)
# 3. Incompatible types from API: update the interface or use a type assertion (sparingly)
```

Never use `// @ts-ignore` or `as any` as a fix — add it as a tech debt ticket if
truly unavoidable.

---

### Test failures — Next.js (Jest / Vitest)

**Problem:** `npm run test` fails in CI but passes locally.

**Cause — environment variables missing in CI:**

Add required env vars to the workflow file or GitHub Actions variables. Check
`.env.example` for required keys.

**Cause — snapshot mismatch:**

```bash
# Update snapshots locally
npm run test -- --updateSnapshot
git add . && git commit -m "test: update snapshots"
```

**Cause — timing / async issues:**

Look for `waitFor` or `act` missing around async interactions in tests. CI runs faster
(no cache warm-up) and exposes timing bugs.

---

### Test failures — Python (pytest)

**Problem:** `make test` fails in CI.

**Cause — import errors / missing dependencies:**

```bash
# Ensure all deps are installed
pip install -r requirements.txt -r requirements-dev.txt

# Check if a new requirement was added without updating requirements.txt
pip freeze | diff - requirements.txt
```

**Cause — moto mock not covering all AWS calls:**

Ensure every AWS service call in the function under test is mocked. Missing mocks
will attempt real AWS calls in CI (which fail due to no credentials).

```python
# Pattern: always decorate test with the services you use
@mock_aws
def test_my_lambda():
    ...
```

---

### Test failures — Flutter

**Problem:** `flutter test` fails in CI.

**Cause — missing golden files:**

```bash
# Update golden files locally
flutter test --update-goldens
git add test/ && git commit -m "test: update golden files"
```

**Cause — platform-specific plugin:**

CI runs in a Linux environment without platform plugins. Mock plugin dependencies
in test setup using `TestWidgetsFlutterBinding` or package mocks.

---

### Build errors — Next.js

**Problem:** `npm run build` fails in CI with module not found or build error.

**Cause — environment variable missing at build time:**

Check `next.config.js` for `NEXT_PUBLIC_*` vars. These must be present at build time.
Add them as GitHub Actions variables (not secrets, as they're embedded in the bundle).

**Cause — dynamic import issue:**

```
Error: Module not found: Can't resolve 'fs'
```

An import that uses Node.js built-ins is being pulled into the client bundle.
Add `"use server"` or move the import behind a dynamic `import()` with `{ ssr: false }`.

---

### Build errors — SAM

**Problem:** `sam build` fails.

```bash
# See full error output
sam build --debug

# Common: requirements.txt not found in function directory
# Check template.yaml — CodeUri must point to the right folder
# Each Lambda function needs its own requirements.txt
```

---

## Deployment Issues

### AWS credential errors in CI

**Problem:** Workflow fails with:
```
Error: Credentials could not be loaded
```
or
```
Not authorized to perform sts:AssumeRoleWithWebIdentity
```

**Cause:** OIDC trust policy is misconfigured or `AWS_OIDC_ROLE_ARN` secret is wrong.

**Solution:**

1. Confirm the secret exists: **Settings → Secrets and variables → Actions → `AWS_OIDC_ROLE_ARN`**
2. Confirm the ARN is correct (no trailing spaces):
   ```bash
   aws iam get-role --role-name github-actions-<repo-name> --query 'Role.Arn'
   ```
3. Check the IAM role's trust policy allows the correct repo and branch:
   ```json
   {
     "StringLike": {
       "token.actions.githubusercontent.com:sub":
         "repo:dateantechnology/<repo-name>:ref:refs/heads/*"
     }
   }
   ```
4. If the role or trust policy needs changing, raise a ticket with **platform-leads**.

---

### SAM deploy failures

**Problem:** `sam deploy` fails with a CloudFormation error.

**Cause — stack already exists in a rollback state:**

```bash
# Check stack status
aws cloudformation describe-stacks --stack-name <stack-name> \
  --region eu-west-1 --query 'Stacks[0].StackStatus'

# If status is ROLLBACK_COMPLETE, delete and redeploy
aws cloudformation delete-stack --stack-name <stack-name> --region eu-west-1
# Wait for deletion, then re-run the deploy workflow
```

**Cause — IAM permissions:** The OIDC role may lack permissions for a new resource type.
Raise with **platform-leads** with the exact error message.

**Cause — parameter mismatch:**

```
Parameter validation failed: Missing required parameter: ...
```

Check `template.yaml` for new `Parameters` blocks — add corresponding values to the
GitHub Environment variables.

---

### S3 sync issues

**Problem:** Files aren't appearing on the CDN after a frontend deployment.

**Cause — S3 sync succeeded but CloudFront cache not invalidated:**

Check the deployment workflow — there should be a step:
```yaml
- name: Invalidate CloudFront
  run: |
    aws cloudfront create-invalidation \
      --distribution-id ${{ vars.CLOUDFRONT_DISTRIBUTION_ID }} \
      --paths "/*"
```

If this step is missing or failed, run it manually:
```bash
aws cloudfront create-invalidation \
  --distribution-id <distribution-id> \
  --paths "/*" \
  --region eu-west-1
```

**Cause — wrong S3 bucket targeted:** Verify `S3_BUCKET_NAME` in GitHub Environment
variables matches the actual bucket name in AWS.

---

### CloudFront returning stale/wrong content

**Problem:** Production shows old content despite a successful deployment.

**Solution (in order):**

1. Hard-refresh the browser: `Ctrl+Shift+R` / `Cmd+Shift+R`
2. Check CloudFront invalidation status:
   ```bash
   aws cloudfront list-invalidations \
     --distribution-id <distribution-id> \
     --query 'InvalidationList.Items[0]'
   ```
3. If status is `InProgress`, wait — invalidations take 1–5 minutes globally.
4. If no recent invalidation exists, trigger one manually (see above).

---

## Local Dev Issues

### Node version mismatch

**Problem:**
```
error: The engine "node" is incompatible with this module. Expected: ">=20"
```

**Cause:** Running wrong Node version.

**Solution:**
```bash
nvm use 20
node --version   # confirm v20.x.x
npm install      # reinstall with correct Node
```

If nvm isn't installed, see [onboarding.md](onboarding.md#core-tools).

---

### Python virtual environment issues

**Problem:** Imports fail, wrong Python version, or `pip install` errors.

**Solution:**
```bash
# Recreate the virtual environment from scratch
deactivate   # if currently active
rm -rf .venv
python --version   # must be 3.12.x — if not, fix pyenv first

python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt -r requirements-dev.txt
```

**Problem:** `ModuleNotFoundError` for a package that's in requirements.txt.

**Cause:** venv not activated, or installed in the wrong environment.

```bash
which python   # should point to .venv/bin/python
source .venv/bin/activate
pip install -r requirements.txt
```

---

### Flutter pub get failures

**Problem:**
```
Because X depends on Y >=2.0.0 which doesn't match any versions, version solving failed.
```

**Solution:**
```bash
# Clear cached packages and retry
flutter pub cache repair
flutter pub get

# If still failing — there may be a version conflict in pubspec.yaml
# Run the resolver in verbose mode to see what's conflicting
flutter pub deps
```

**Problem:** `flutter doctor` shows issues with Xcode or Android SDK.

Follow the exact instructions `flutter doctor` prints — it links to the fix for each issue.
Common fixes:
- Xcode: `sudo xcode-select --switch /Applications/Xcode.app`
- Android licenses: `flutter doctor --android-licenses`

---

### SAM local API issues

**Problem:** `sam local start-api` fails to start.

**Cause — Docker not running:**
```bash
docker info   # will error if Docker isn't running
open -a Docker   # macOS: start Docker Desktop
```

**Cause — port already in use:**
```bash
lsof -i :3001   # find what's using the port
kill <PID>
sam local start-api --port 3001 --env-vars .env
```

**Cause — Lambda cold-start timeout in local mode:**

SAM local has a longer cold start than real Lambda. If your request times out locally
but succeeds in CI, increase the local timeout:
```bash
sam local start-api --warm-containers EAGER
```

**Problem:** Environment variables not picked up by the Lambda.

Ensure your `.env` file is in the format:
```json
{
  "FunctionName": {
    "MY_VAR": "value"
  }
}
```
(SAM local uses JSON format, not shell `.env` format for `--env-vars`.)

---

## GitHub Issues

### PR blocked — required checks haven't run

**Problem:** "Required status checks" section shows checks as "Expected" but they
never ran.

**Cause:** The CI workflow is only triggered on certain events, and your PR didn't
match.

**Solution:**
```bash
# Push an empty commit to re-trigger CI
git commit --allow-empty -m "ci: re-trigger checks"
git push
```

If checks still don't appear, verify the workflow file has `pull_request` in the
`on:` trigger:
```yaml
on:
  push:
    branches: [main, env/test, env/prod]
  pull_request:
    branches: [main]
```

---

### PR blocked — signed commits required

**Problem:** PR is blocked because some commits are not signed.

**Cause:** Commit signing was not enabled when the commit was made.

**Solution:**
```bash
# Enable signing
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# Amend the last commit to sign it
git commit --amend --no-edit -S

# Or, for multiple commits — interactive rebase
git rebase -i origin/main
# Mark commits as 'edit', then for each:
git commit --amend --no-edit -S
git rebase --continue

git push --force-with-lease
```

---

### Dependabot PR merge conflicts

**Problem:** Dependabot raised a PR but it now has a merge conflict.

**Solution:**

1. Leave a comment on the Dependabot PR: `@dependabot rebase`
2. Dependabot will automatically rebase the PR against `main`.
3. If it doesn't respond within a few minutes, close and re-open the PR to trigger it.

For major version bumps with real conflicts:
```bash
gh pr checkout <pr-number>
git rebase origin/main
# Resolve conflicts manually
git add .
git rebase --continue
git push --force-with-lease
```

---

### "You need to resolve all conversations" blocking merge

**Problem:** PR shows unresolved review conversations but you've addressed them all.

**Solution:**

The original reviewer must mark conversations as resolved, or you must click
**"Resolve conversation"** on each thread yourself (if the reviewer has approved
and you've addressed their comments).

Don't resolve conversations that are still genuinely unresolved — push a fix first.

---

> 🆘 **Still stuck?** Check `#engineering-help` in Slack first. If it's infrastructure
> or CI-related, tag **@platform-leads**. For security issues, email
> mercy.akinkuotu@hotmail.com directly.
```

---

All 8 files are complete. Here's a summary of what was produced:

| File | Purpose | Key highlights |
|------|---------|----------------|
| `profile/README.md` | Org landing page | Tech stack badges, team table, resource links, onboarding pointer |
| `CONTRIBUTING.md` | Org-wide contribution standard | Branch table, Conventional Commits guide, PR process, review etiquette |
| `CODE_OF_CONDUCT.md` | Contributor Covenant v2.1 adaptation | Professional tone, enforcement tiers, reporting contact |
| `SECURITY.md` | Vulnerability disclosure policy | Supported versions, response SLAs (48h ack / 7d fix plan), coordinated disclosure |
| `docs/onboarding.md` | New joiner guide | Tool install steps (nvm, pyenv), GitHub/AWS setup, per-stack dev server, first-day checklist |
| `docs/how-to-create-a-new-service.md` | New repo bootstrap guide | Naming conventions, 9-step setup, OIDC config, branch protection, CODEOWNERS |
| `docs/workflow.md` | Day-to-day dev workflow | Mermaid flow diagram, full ticket→production lifecycle, env branch deploy strategy |
| `docs/troubleshooting.md` | Problem→Cause→Solution reference | 20+ scenarios across CI, deployment, local dev (all 3 stacks), and GitHub |
