# Security

This repository is maintained by **Nexus Communications Technology** (Nexuscomm LLC).

## Reporting a security issue

If you discover a security vulnerability, **do not** open a public issue.

Instead, email **office@nexusct.com** with:
- A description of the vulnerability
- Steps to reproduce (if applicable)
- The commit/branch where you observed it

We will acknowledge within 2 business days.

## What belongs in this repo

**Never commit:**
- Passwords, API keys, tokens, private keys, certificates
- Internal pricing data, dealer costs, margin percentages
- Customer PII or PHI
- Internal network addresses or credentials
- Database dumps or backups containing real data

Use environment variables, `config.js` (gitignored), or a runtime
config endpoint — never hardcode.

## Pre-commit hooks

This repo uses [pre-commit](https://pre-commit.com/) with:
- `gitleaks` — detects committed secrets
- `detect-secrets` — second-layer scanner
- Nexus-specific pattern blocks — rejects known-bad strings

**Setup (one-time per clone):**

```bash
pip install pre-commit
pre-commit install
```

Hooks will run automatically on every commit.

## Incident history

- **2026-08-15** — Secret scanning alert remediation (NCT-SEC-2026-08-15-001).
  **Issue**: Google Maps API key `AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk` remains
  in git history at commit `a0d76c7` within `.pre-commit-config.yaml`.
  **Status**: Key removed from HEAD (commit `b71f5da`) but still present in history.
  **Remediation**: Git history must be rewritten using `git-filter-repo` or BFG Repo-Cleaner.
  **Action required**: Owner must rotate the exposed API key and force-push cleaned history.
  See below for detailed rotation and history-cleaning instructions.

- **2026-04-19** — Credential exposure incident (NCT-SEC-2026-04-19-001).
  Rotated: Google Maps API key, admin password. Full report on file.

## Git History Remediation

### Removing secrets from git history

When a secret is committed to git, removing it from the current commit (HEAD) is **not sufficient**.
The secret remains in git history and can be accessed by anyone who clones the repository.

To properly remove secrets from history:

#### Option 1: Using git-filter-repo (recommended)

1. **Install git-filter-repo**:
   ```bash
   pip install git-filter-repo
   ```

2. **Create a replacement file** (`/tmp/secret-replacements.txt`):
   ```
   AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk==>REDACTED_GOOGLE_API_KEY
   ```

3. **Run git-filter-repo**:
   ```bash
   cd /path/to/nexus-plugin
   git-filter-repo --replace-text /tmp/secret-replacements.txt --force
   ```

4. **Re-add the remote** (git-filter-repo removes it):
   ```bash
   git remote add origin https://github.com/nexusct/nexus-plugin.git
   ```

5. **Force-push to remote** (⚠️ WARNING: This rewrites history):
   ```bash
   git push origin --force --all
   git push origin --force --tags
   ```

6. **Notify all team members** to delete and re-clone the repository.

#### Option 2: Using BFG Repo-Cleaner

1. **Download BFG**: https://rtyley.github.io/bfg-repo-cleaner/
2. **Run BFG**:
   ```bash
   bfg --replace-text /tmp/secret-replacements.txt nexus-plugin.git
   ```
3. **Follow force-push steps above**

### After history is cleaned

- GitHub secret scanning alert should automatically close
- All team members must re-clone (old clones still have the secret)
- Update any CI/CD or automation that references commit SHAs

## API Key Rotation Instructions

### If a Google Maps API key is exposed:

1. **Immediately restrict the exposed key** in Google Cloud Console:
   - Go to https://console.cloud.google.com/apis/credentials
   - Find the exposed API key: `AIzaSyAd72xUaF049-dbkwTAfSvsjQhmp9YLDpk`
   - Click "Edit" → Add application restrictions and API restrictions
   - Or delete the key if it's no longer in use

2. **Create a new API key**:
   - In Google Cloud Console, click "Create Credentials" → "API key"
   - Restrict the new key to specific APIs (e.g., Maps JavaScript API, Places API)
   - Add HTTP referrer restrictions or IP address restrictions
   - Copy the new key

3. **Update your server configuration**:
   ```bash
   # On your production/staging servers
   export GOOGLE_MAPS_API_KEY="your-new-api-key-here"
   # Or update config.js / config.local.js
   ```

4. **Verify the new key works**:
   - Test all functionality that uses the Google Maps API
   - Check server logs for API errors

5. **Delete the old exposed key** from Google Cloud Console after verification.

### General secret rotation workflow:

1. Generate new credential
2. Update all systems that use it (servers, CI/CD, etc.)
3. Test thoroughly
4. Revoke/delete the old credential
5. Document the rotation in this file with date and reference number
