# GitHub Security Audit Report

**Date:** 2026-03-09
**Scope:** All public repositories across bgilly personal account and 5 GitHub organizations
**Auditor:** Automated scan via Claude Code
**Objective:** Identify leaked secrets, API keys, tokens, passwords, credentials, and other sensitive data in public-facing repositories

---

## Executive Summary

**No leaked secrets, credentials, or sensitive data were found across any public repository.**

16 public repositories were audited across 6 GitHub accounts/organizations. Three organizations (drinkpoint, StartingStrengthGyms, Considered-Systems) had zero public repositories. All speakup repos and bgilly personal repos are unmodified forks with no custom commits, presenting minimal risk. The 4 original repositories under Family-IT-Guy follow security best practices with proper credential management.

---

## Scope Overview

| Owner | Public Repos | Type | Status |
|-------|-------------|------|--------|
| **bgilly** | 3 | All forks | CLEAN |
| **drinkpoint** (MenuPoint) | 0 | N/A | N/A - No public repos |
| **speakup** (SpeakUp) | 8 | All forks | CLEAN |
| **StartingStrengthGyms** | 0 | N/A | N/A - No public repos |
| **Considered-Systems** | 0 | N/A | N/A - No public repos |
| **Family-IT-Guy** | 5 | 4 original, 1 fork | CLEAN |

---

## Detailed Findings by Account/Organization

### 1. bgilly (Personal Account)

#### bgilly/experiments (fork of SWE-bench/experiments)
- **Status:** CLEAN
- **Custom commits by bgilly:** None (unmodified fork)
- **Notes:** References external `keys.cfg` for OpenAI API keys, but this file is properly listed in `.gitignore` and NOT committed. `download_logs.py` uses `boto3` for AWS S3 but loads credentials from environment, not hardcoded.

#### bgilly/fastmail-cli (fork of radiosilence/fastmail-cli)
- **Status:** CLEAN
- **Custom commits by bgilly:** None (unmodified fork)
- **Notes:** Credentials handled properly throughout:
  - Tokens loaded from env vars (`FASTMAIL_API_TOKEN`, `FASTMAIL_APP_PASSWORD`) or config file
  - Config file uses restricted permissions (0o600)
  - `.gitignore` excludes `.env` and `*.local.*` files
  - CI workflow uses only built-in `GITHUB_TOKEN`
  - Test data uses placeholders (`"test-token"`, `"test@example.com"`)
  - README examples use safe placeholders (`"fmu1-..."`, `"your-token-here"`)
  - Git history clean — no secrets ever committed

#### bgilly/instance_112 (fork of django/django)
- **Status:** CLEAN
- **Custom commits by bgilly:** None (unmodified fork)
- **Notes:** Standard Django fork. `SECRET_KEY = ""` (intentionally empty default). No custom settings or credential files.

---

### 2. drinkpoint (MenuPoint)
- **No public repositories.** Nothing to audit.

---

### 3. speakup (SpeakUp)

All 8 repositories are unmodified forks with no custom commits from speakup members. Commit histories confirmed all changes are from upstream contributors.

| Repository | Upstream | Custom Commits | Secrets Found |
|-----------|----------|---------------|---------------|
| nodejs-cookbook | mdxp/nodejs-cookbook | None | None |
| chef-mongodb | edelight/chef-mongodb | None | None |
| redisio | sous-chefs/redisio | None | None |
| chef-logstash | lusis/chef-logstash | None | None |
| vagrant-centos | — | None | None |
| textAngular | textAngular/textAngular | None | None |
| js-emoji | iamcal/js-emoji | None | None |
| grunt-inline-css | jgallen23/grunt-inline-css | None | None |

**Chef cookbook attributes reviewed:** MongoDB key files set to `nil`, Redis `requirepass` set to `nil`, Logstash attributes contain no credentials. All follow standard patterns of deferring sensitive values to encrypted data bags or environment configuration.

---

### 4. StartingStrengthGyms
- **No public repositories.** Nothing to audit.

---

### 5. Considered-Systems
- **No public repositories.** Nothing to audit.

---

### 6. Family-IT-Guy

#### Family-IT-Guy/perplexity-mcp (ORIGINAL — High Priority)
- **Status:** CLEAN
- **Description:** MCP server integrating Perplexity AI with Claude Desktop
- **Credential handling:**
  - API key loaded via `process.env.PERPLEXITY_API_KEY` — never hardcoded
  - Throws error if key is missing: `"PERPLEXITY_API_KEY is required"`
  - `.gitignore` properly excludes `.env`, `node_modules/`, editor files
  - `package.json` contains no suspicious scripts or hardcoded values
  - All source files (`index.ts`, `perplexity.ts`, `research-store.ts`, `synthesis.ts`) pass credential management via constructors and env vars
  - No Perplexity keys (`pplx-...`), Anthropic keys (`sk-ant-...`), or OpenAI keys (`sk-...`) found

#### Family-IT-Guy/LLM-Rules (ORIGINAL)
- **Status:** CLEAN
- **Description:** LLM rules and frameworks (markdown files only)
- **Notes:** Contains only markdown documentation files (`rules/`, `frameworks/`, `README.md`). No code, no configuration files, no credentials. Pure documentation repo.

#### Family-IT-Guy/claude-code-skills (ORIGINAL)
- **Status:** CLEAN
- **Description:** Claude Code skills for AI development
- **Credential handling:**
  - `.gitignore` excludes `api-key.env` files
  - Uses `api-key.env.example` template pattern (example file only, no real keys)
  - README directs users to configure API keys separately in `~/.claude/` directory
  - Skills stored as markdown instruction files, no embedded credentials

#### Family-IT-Guy/claude-code-starter-kit (ORIGINAL)
- **Status:** CLEAN
- **Description:** Session recovery and self-improvement system for Claude Code CLI
- **Notes:** Contains only YAML configs, markdown workflows, and a shell script. No API keys, tokens, or credentials found. Configuration references `~/.claude/` directory for user-specific settings. Shell script (`statusline-script.sh`) processes JSON input dynamically with no hardcoded secrets.

#### Family-IT-Guy/experiments (fork of SWE-bench/experiments)
- **Status:** CLEAN
- **Custom commits:** None (unmodified fork)
- **Notes:** Same upstream as bgilly/experiments. No custom modifications.

---

## Methodology

1. **Repository enumeration** via GitHub API (`/users/{user}/repos` and `/orgs/{org}/repos`)
2. **File tree analysis** — identified suspicious filenames (`.env`, `*secret*`, `*credential*`, `*key*`, config files)
3. **Source code review** — searched for patterns: API keys, tokens, passwords, connection strings, SSH keys, AWS/GCP/Azure credentials
4. **Git history analysis** — searched commit history for previously committed and removed secrets
5. **Commit authorship verification** — confirmed which repos have custom commits vs. unmodified forks
6. **CI/CD pipeline review** — checked GitHub Actions workflows for exposed secrets
7. **Gitignore verification** — confirmed sensitive file patterns are properly excluded

### Patterns searched:
- `api_key`, `api_token`, `password`, `secret`, `token`, `bearer`
- `sk-` (OpenAI), `sk-ant-` (Anthropic), `pplx-` (Perplexity), `fmu1-` (Fastmail)
- `PRIVATE_KEY`, `AWS_SECRET`, `AWS_ACCESS_KEY`
- `.env`, `credentials.json`, `secrets.yaml`, `*.pem`, `*.key`

---

## Recommendations

1. **Continue current practices** — All original repos follow security best practices (env vars, .gitignore, placeholder examples)
2. **Consider archiving stale forks** — The 11 unmodified forks (speakup org especially, dating to 2011-2015) serve no apparent purpose and increase surface area. Consider making them private or deleting them.
3. **Enable GitHub secret scanning** — If not already enabled, turn on GitHub's built-in secret scanning alerts for all organizations
4. **Enable branch protection** — Ensure main/master branches have protection rules to prevent accidental secret commits
5. **Periodic audits** — Re-run this audit periodically, especially after new repos are created or significant commits are made

---

## Summary

| Category | Count |
|----------|-------|
| Total accounts/orgs audited | 6 |
| Total public repos audited | 16 |
| Repos with leaked secrets | **0** |
| Original repos (highest risk) | 4 |
| Unmodified forks (low risk) | 12 |
| Orgs with no public repos | 3 |
