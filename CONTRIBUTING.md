# Contributing to Datean Technology

This guide applies to every repository under the **dateantechnology** GitHub organisation
that does not carry its own `CONTRIBUTING.md`. Read it once; refer back as needed.

---

## Table of Contents

1. [Branching Strategy](#branching-strategy)
2. [Commit Message Format](#commit-message-format)
3. [Opening a Pull Request](#opening-a-pull-request)
4. [What Makes a Good PR Description](#what-makes-a-good-pr-description)
5. [Code Review Etiquette](#code-review-etiquette)
6. [Merge Strategy](#merge-strategy)

---

## Branching Strategy

| Branch | Purpose | Protection | Auto-deploy |
|--------|---------|------------|-------------|
| `main` | Source of truth — always deployable | PR + 1 approval, signed commits required | ❌ |
| `env/test` | Test environment | PR required | ✅ → Test |
| `env/prod` | Production environment | PR + 1 approval required | ✅ → Production |
| `feat/<ticket>-<slug>` | New features | None | ❌ |
| `fix/<ticket>-<slug>` | Bug fixes | None | ❌ |
| `chore/<ticket>-<slug>` | Dependency updates, config, tooling | None | ❌ |

### Branch naming examples

```
feat/DT-142-user-auth-flow
fix/DT-209-broken-checkout-redirect
chore/DT-301-bump-next-to-14
```

**Rules:**
- Always branch from `main` unless explicitly told otherwise.
- Keep branches short-lived — open a PR within a day or two.
- Delete the branch after merge (enforced automatically on all repos).

---

## Commit Message Format

We follow **[Conventional Commits](https://www.conventionalcommits.org/)**.

```
<type>(<optional scope>): <short imperative description>

[optional body — wrap at 72 chars]

[optional footer(s): BREAKING CHANGE, Closes #123]
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | A new feature visible to users or consumers |
| `fix` | A bug fix |
| `chore` | Tooling, dependencies, config — no production logic change |
| `docs` | Documentation only |
| `refactor` | Code restructure with no behaviour change |
| `test` | Adding or fixing tests |
| `ci` | Changes to GitHub Actions or deployment config |
| `perf` | Performance improvement |

### Examples

```bash
# Good
feat(auth): add JWT refresh token rotation
fix(checkout): resolve redirect loop on expired session
chore: bump aws-sam-cli to 1.115.0
docs: update onboarding guide with Flutter 3.22 steps
ci: add OIDC role assumption to prod workflow
refactor(api): extract payment handler into separate module

# Bad — avoid these patterns
git commit -m "fix stuff"
git commit -m "WIP"
git commit -m "Update"
git commit -m "feat: Added the new thing for the user to do the login"
```

**Signed commits are required on `main`.** Configure GPG or SSH signing:

```bash
# SSH signing (recommended)
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

---

## Opening a Pull Request

Follow these steps every time:

1. **Ensure your branch is up to date with `main`:**
   ```bash
   git fetch origin
   git rebase origin/main
   ```

2. **Run the full local check suite before pushing:**
   ```bash
   # Next.js repos
   npm run lint && npm run typecheck && npm run test

   # Python SAM repos
   make lint && make test

   # Flutter
   flutter analyze && flutter test
   ```

3. **Push and open the PR against `main`:**
   ```bash
   git push -u origin feat/DT-142-user-auth-flow
   gh pr create --base main --fill
   ```

4. **Fill in the PR description** (see [below](#what-makes-a-good-pr-description)).

5. **Assign at least one reviewer** from the relevant team (see `CODEOWNERS`).

6. **Link the issue/ticket:**
   Add `Closes #<issue-number>` or reference the project ticket in the description.

7. **Do not merge your own PR** unless you are the sole maintainer of that service.

---

## What Makes a Good PR Description

Use this template (most repos include it as `.github/PULL_REQUEST_TEMPLATE.md`):

```markdown
## Summary
<!-- 2–3 sentences: what does this PR do and why? -->

## Changes
<!-- Bullet list of concrete changes made -->
-
-

## How to Test
<!-- Steps to verify the change works. Be specific. -->
1.
2.

## Screenshots / Recordings
<!-- For UI changes — before/after if possible -->

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated if behaviour changed
- [ ] No hardcoded secrets or credentials
- [ ] Conventional commit messages throughout
- [ ] Linked ticket: #___
```

**A good PR description:**
- Answers *why*, not just *what*
- Makes the reviewer's job faster, not slower
- Calls out anything risky or uncertain with a ⚠️
- Keeps scope tight — one concern per PR

---

## Code Review Etiquette

### For authors

- Keep PRs small. Aim for < 400 lines of diff. Large PRs get worse reviews.
- Respond to every comment, even if just to say you've addressed it.
- Don't silently update code after a "looks good" — flag any post-approval changes.
- Mark your own nits as `[nit]` so reviewers can skip them if under time pressure.

### For reviewers

- **Respond within one working day.** If you can't, say so.
- Be specific. "This looks wrong" is not a review comment.
- Distinguish blockers from suggestions:
  - `[blocker]` — must be fixed before merge
  - `[suggestion]` — take it or leave it
  - `[nit]` — minor style, totally optional
- Review the logic and intent, not just the syntax.
- Approve explicitly when satisfied. Don't leave PRs in limbo.

### Tone

Reviews are about code, not people. Be direct, be kind.

```
# Good
[suggestion] This could be extracted into a helper — would make it easier to test in isolation.

# Not good
Why did you do it this way? This is obviously wrong.
```

---

## Merge Strategy

**All repositories use squash merge exclusively.**

- Every PR becomes a single commit on `main` (or the target branch).
- The squash commit title must be a valid Conventional Commit message.
- GitHub enforces this — rebase and merge, and merge commits, are disabled.

After merge:
- The source branch is **automatically deleted**.
- If the target branch is `env/test` or `env/prod`, a deployment workflow fires automatically.
- Monitor the Actions tab to confirm the deployment completes.
```

---
