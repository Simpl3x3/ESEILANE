# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| 1.x (latest) | Yes |
| < 1.0 | No |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues.**

To report a security vulnerability, please:

1. Go to the [Security tab](https://github.com/Simpl3x3/ESEILANE/security) of this repository.
2. Click **"Report a vulnerability"**.
3. Fill in the details of the vulnerability.

We aim to acknowledge reports within **48 hours** and provide a fix within **7 days** for critical issues.

Please include:
- A description of the vulnerability and its potential impact
- Steps to reproduce the issue
- Any relevant code, logs, or screenshots
- Your GitHub handle (for credit in the release notes, if desired)

## Security Best Practices

When self-hosting ESEILANE:

- Always run behind a reverse proxy with TLS (HTTPS)
- Use strong, unique passwords for all admin accounts
- Enable RBAC and grant minimum required permissions
- Keep ESEILANE and all dependencies up to date
- Monitor access logs for unusual activity
- Use Docker with non-root users in production

Thank you for helping keep ESEILANE and its users safe.
