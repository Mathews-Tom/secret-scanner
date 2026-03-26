# Secret Remediation Guide

Step-by-step procedures for handling detected secrets.

## Immediate Response Workflow

```
1. REMOVE secret from source code
2. ROTATE the compromised credential
3. AUDIT git history for prior exposure
4. VERIFY rotation succeeded
5. PREVENT recurrence (.gitignore, pre-commit hooks)
```

## Removal Methods

### From Current Code
Replace hardcoded secrets with environment variable references:

| Language | Before | After |
|----------|--------|-------|
| Python | `API_KEY = "sk_live_..."` | `API_KEY = os.environ["API_KEY"]` |
| Node.js | `const key = "sk_live_..."` | `const key = process.env.API_KEY` |
| Go | `key := "sk_live_..."` | `key := os.Getenv("API_KEY")` |
| Java | `String key = "sk_live_..."` | `String key = System.getenv("API_KEY")` |
| Ruby | `key = "sk_live_..."` | `key = ENV.fetch("API_KEY")` |

### From Git History
```bash
# Check if secret exists in history
git log -p -S 'SECRET_PATTERN' --all

# If found, history rewrite required:
# Option 1: git-filter-repo (preferred)
git filter-repo --blob-callback '
  blob.data = blob.data.replace(b"SECRET_VALUE", b"REDACTED")
'

# Option 2: BFG Repo Cleaner
bfg --replace-text passwords.txt repo.git

# Force push after rewrite (coordinate with team)
git push --force --all
```

## Credential Rotation by Provider

### AWS
1. Go to IAM Console > Users > Security Credentials
2. Create new access key
3. Update all systems using the old key
4. Deactivate old key, monitor for failures
5. Delete old key after 24-48 hours

### GitHub
1. Settings > Developer settings > Personal access tokens
2. Revoke compromised token
3. Generate new token with minimum required scopes
4. Update CI/CD and integrations

### Stripe
1. Stripe Dashboard > Developers > API keys
2. Roll the secret key (generates new, invalidates old)
3. Update server-side integrations
4. Webhook secrets: roll in Dashboard > Webhooks > Signing secret

### Slack
1. api.slack.com > Your Apps > OAuth & Permissions
2. Revoke compromised token
3. Reinstall app to generate new tokens
4. Update bot deployments

### Google Cloud
1. Cloud Console > APIs & Services > Credentials
2. Delete compromised API key
3. Create new key with appropriate restrictions
4. Update applications

### Database Credentials
1. Change password on database server
2. Update connection strings in secrets manager
3. Restart application instances
4. Verify connectivity

## Prevention Measures

### .gitignore Entries
```gitignore
.env
.env.local
.env.*.local
*.pem
*.key
*.p12
*.pfx
credentials.json
service-account.json
```

### Pre-commit Hook Setup
```bash
# Using pre-commit framework
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

### Environment Variable Management

| Environment | Tool | Notes |
|-------------|------|-------|
| Local dev | `.env` file + dotenv | Never commit .env |
| CI/CD | Pipeline secrets | GitHub Actions secrets, GitLab CI variables |
| Staging | Secrets manager | AWS SSM, Vault, GCP Secret Manager |
| Production | Secrets manager | Same as staging, separate credentials |

## Exposure Assessment

### Questions to Answer After Detection
1. Was the secret ever pushed to a remote repository?
2. Is the repository public or private?
3. How long was the secret exposed in history?
4. What access does this credential grant?
5. Are there audit logs showing unauthorized use?

### Risk Matrix

| Repository Visibility | Duration | Action |
|----------------------|----------|--------|
| Public | Any | Rotate immediately, check audit logs |
| Private (team) | > 24 hours | Rotate, review access logs |
| Private (personal) | < 1 hour | Rotate if pushed to remote |
| Never pushed | Any | Remove from code, rotation optional |

## Verification Checklist
- [ ] Secret removed from current code
- [ ] Environment variable reference in place
- [ ] Old credential rotated/revoked
- [ ] Git history cleaned (if pushed to remote)
- [ ] .gitignore updated for file type
- [ ] Pre-commit hook installed
- [ ] No unauthorized usage in audit logs
