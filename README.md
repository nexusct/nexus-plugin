# nexus-plugin

Nexus Plugin repository maintained by Nexus Communications Technology (Nexuscomm LLC).

## Overview

This repository contains plugin configurations and tooling for Nexus services. It implements security best practices and automated secret detection to prevent credential leaks.

## Setup

### Prerequisites

- Git
- Python 3.7+ (for pre-commit hooks)
- Node.js (if applicable for plugins)

### Installation

1. Clone the repository:

```bash
git clone https://github.com/nexusct/nexus-plugin.git
cd nexus-plugin
```

2. Install pre-commit hooks (required):

```bash
pip install pre-commit
pre-commit install
```

3. Copy environment template (if applicable):

```bash
cp .env.example .env
```

4. Configure your local environment variables in `.env` (never commit this file).

## Security

This repository implements multiple layers of security protection:

- **Pre-commit hooks**: Automated secret detection using gitleaks and detect-secrets
- **Pattern blocking**: Prevents known-bad strings from being committed
- **Gitignore rules**: Blocks credentials, private keys, and sensitive configuration files

See [SECURITY.md](SECURITY.md) for:
- How to report security vulnerabilities
- What should never be committed
- Pre-commit hook setup instructions
- Incident history and remediation steps

**IMPORTANT**: Never commit API keys, passwords, tokens, or other secrets. Use environment variables or server-side configuration files instead.

## Configuration

Configuration files that may contain secrets should:
- Be listed in `.gitignore` (e.g., `config.js`, `config.local.js`)
- Use environment variables for sensitive values
- Have `.example` templates checked into the repository

Example:
```javascript
// config.js (gitignored, not committed)
module.exports = {
  googleMapsApiKey: process.env.GOOGLE_MAPS_API_KEY,
  // ... other config
};
```

## Contributing

1. Ensure pre-commit hooks are installed
2. Never commit secrets or credentials
3. Run `pre-commit run --all-files` before important pushes
4. Follow the security guidelines in [SECURITY.md](SECURITY.md)

## License

Proprietary - Nexus Communications Technology (Nexuscomm LLC)

## Contact

- **Security issues**: office@nexusct.com (see [SECURITY.md](SECURITY.md))
- **General inquiries**: office@nexusct.com
