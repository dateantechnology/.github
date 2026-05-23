# Developer Onboarding Guide

Welcome to Datean Technology. This guide gets you from zero to a running local
environment across all our stacks. Work through it top to bottom on your first day.

---

## Table of Contents

1. [Prerequisites — Tools to Install](#1-prerequisites--tools-to-install)
2. [GitHub Access Setup](#2-github-access-setup)
3. [Local Dev Setup by Stack](#3-local-dev-setup-by-stack)
   - [Next.js (TypeScript)](#31-nextjs-typescript)
   - [Python AWS SAM (Lambda)](#32-python-aws-sam-lambda)
   - [Flutter (Mobile)](#33-flutter-mobile)
4. [First Day Checklist](#4-first-day-checklist)
5. [Useful Links](#5-useful-links)
6. [Who to Ask for What](#6-who-to-ask-for-what)

---

## 1. Prerequisites — Tools to Install

Install everything in this section before doing anything else.
All version numbers are minimums unless marked exact.

### Core tools

| Tool | Version | Install |
|------|---------|---------|
| **Git** | ≥ 2.43 | `brew install git` / [git-scm.com](https://git-scm.com/) |
| **Node.js** | 20 LTS (exact) | Use `nvm` — see below |
| **Python** | 3.12 (exact) | Use `pyenv` — see below |
| **Flutter** | ≥ 3.x | [flutter.dev/docs/get-started/install](https://docs.flutter.dev/get-started/install) |
| **AWS CLI** | ≥ 2.15 | [docs.aws.amazon.com/cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| **AWS SAM CLI** | ≥ 1.115 | `brew tap aws/tap && brew install aws-sam-cli` |
| **Docker** | ≥ 25 | [docker.com/get-started](https://www.docker.com/get-started/) |
| **gh CLI** | ≥ 2.47 | `brew install gh` |

### Node.js via nvm (recommended)

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc   # or ~/.zshrc

# Install and pin Node 20
nvm install 20
nvm alias default 20
node --version   # should print v20.x.x
```

### Python via pyenv (recommended)

```bash
# Install pyenv
brew install pyenv

# Add to shell (zsh shown — adjust for bash)
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
source ~/.zshrc

# Install and set Python 3.12
pyenv install 3.12
pyenv global 3.12
python --version   # should print Python 3.12.x
```

---

## 2. GitHub Access Setup

### Step 1 — Enable 2FA

Two-factor authentication is **mandatory** for all members of the dateantechnology org.

1. Go to **GitHub → Settings → Password and authentication**
2. Enable 2FA using an authenticator app (Authy, 1Password, Google Authenticator)
3. Save your recovery codes somewhere safe

### Step 2 — Accept the org invitation

Check your email or visit [github.com/orgs/dateantechnology](https://github.com/orgs/dateantechnology)
and accept the pending invitation. If you don't see one, ask your team lead.

### Step 3 — Configure Git identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@dateantechnology.com"
```

### Step 4 — Set up authentication

**Option A: SSH key (recommended)**

```bash
# Generate a key if you don't have one
ssh-keygen -t ed25519 -C "your.email@dateantechnology.com"

# Copy the public key
cat ~/.ssh/id_ed25519.pub

# Add it to GitHub: Settings → SSH and GPG keys → New SSH key
# Test the connection
ssh -T git@github.com
```

**Option B: HTTPS with gh CLI**

```bash
gh auth login
# Follow the prompts — select HTTPS, then authenticate via browser
```

### Step 5 — Set up commit signing (required for `main`)

```bash
# SSH signing
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# Verify a signed commit
git log --show-signature -1
```

### Step 6 — Configure AWS CLI for local development

You will receive an IAM user or SSO configuration from your team lead.

```bash
# If using SSO (recommended)
aws configure sso
# Follow the prompts — use profile name: dateantechnology-dev
# Region: eu-west-1

# Verify access
aws sts get-caller-identity --profile dateantechnology-dev
```

---

## 3. Local Dev Setup by Stack

Clone the repository you need to work on first:

```bash
gh repo clone dateantechnology/<repo-name>
cd <repo-name>
```

---

### 3.1 Next.js (TypeScript)

Applies to all `*-www`, `*-admin`, and `*-portal` repositories.

```bash
# Ensure you're on Node 20
node --version   # v20.x.x

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.local
# Fill in values — ask your team lead for dev secrets

# Start the dev server
npm run dev
# Open http://localhost:3000
```

**Common scripts:**

| Script | Purpose |
|--------|---------|
| `npm run dev` | Start local dev server with hot reload |
| `npm run build` | Production build |
| `npm run lint` | Run ESLint |
| `npm run typecheck` | Run TypeScript compiler (no emit) |
| `npm run test` | Run Jest / Vitest test suite |

---

### 3.2 Python AWS SAM (Lambda)

Applies to all `*-bff`, `*-api`, and `*-ingestor` repositories.

```bash
# Ensure you're on Python 3.12
python --version   # Python 3.12.x

# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt   # dev/test deps

# Copy environment template
cp .env.example .env
# Fill in local values

# Run unit tests
make test
# or: pytest tests/unit

# Start local API with SAM (requires Docker running)
sam build
sam local start-api --env-vars .env --profile dateantechnology-dev
# API available at http://localhost:3001
```

**Common make targets:**

| Target | Purpose |
|--------|---------|
| `make lint` | Run ruff + mypy |
| `make test` | Run pytest |
| `make build` | Run `sam build` |
| `make invoke` | Invoke a specific Lambda function locally |

---

### 3.3 Flutter (Mobile)

```bash
# Verify Flutter is installed correctly
flutter doctor
# Fix any issues it flags before continuing

# Install dependencies
flutter pub get

# Copy environment config if present
cp lib/config/env.example.dart lib/config/env.dart

# Run on a simulator/emulator
flutter run

# Run tests
flutter test

# Static analysis
flutter analyze
```

**Targets:**

| Command | Purpose |
|---------|---------|
| `flutter run` | Run on connected device/emulator |
| `flutter build ios` | Build iOS release |
| `flutter build apk` | Build Android APK |
| `flutter test` | Run unit + widget tests |
| `flutter analyze` | Static analysis |

---

## 4. First Day Checklist

Work through this list and tick each item off:

- [ ] All tools in [Section 1](#1-prerequisites--tools-to-install) installed and version-checked
- [ ] GitHub 2FA enabled
- [ ] Org invitation accepted
- [ ] Git identity configured (`user.name`, `user.email`)
- [ ] SSH key added to GitHub and connection tested
- [ ] Commit signing configured and verified
- [ ] AWS CLI configured — `aws sts get-caller-identity` returns your identity
- [ ] At least one repo cloned and local dev server running
- [ ] Introduced yourself in the team channel
- [ ] Added to the relevant GitHub team by your team lead (ACT / COG-IT / R&DIT)
- [ ] Read [CONTRIBUTING.md](../CONTRIBUTING.md)
- [ ] Read [docs/workflow.md](workflow.md)

---

## 5. Useful Links

| Resource | URL |
|----------|-----|
| GitHub Org | https://github.com/dateantechnology |
| AWS Console (eu-west-1) | https://eu-west-1.console.aws.amazon.com/ |
| Engineering workflow guide | [docs/workflow.md](workflow.md) |
| Creating a new service | [docs/how-to-create-a-new-service.md](how-to-create-a-new-service.md) |
| Troubleshooting | [docs/troubleshooting.md](troubleshooting.md) |
| Conventional Commits spec | https://www.conventionalcommits.org/ |
| AWS SAM docs | https://docs.aws.amazon.com/serverless-application-model/ |

---

## 6. Who to Ask for What

| Topic | Ask |
|-------|-----|
| GitHub access, repo permissions, team membership | Your **team lead** |
| AWS credentials, IAM roles, infrastructure questions | **platform-leads** team |
| Next.js / frontend questions | **ACT** or **COG-IT** (per repo) |
| Python / Lambda / SAM questions | Repo owner (check `CODEOWNERS`) |
| Flutter / mobile questions | **R&DIT** or **COG-IT** (per repo) |
| CI/CD pipeline failures | **platform-leads** team |
| Security concerns | mercy.akinkuotu@hotmail.com |
| Process / workflow questions | `CONTRIBUTING.md` first, then your team lead |

> 💡 **Tip:** Check the `CODEOWNERS` file in any repo to find the primary owner for that service.
```

---
