# Provider Key Patterns Reference

Regex patterns for detecting known provider credentials in source code.

## Cloud Providers

### AWS

| Secret Type | Pattern | Example (Redacted) |
|-------------|---------|-------------------|
| Access Key ID | `AKIA[0-9A-Z]{16}` | `AKIA...XMPL` |
| Secret Access Key | `[A-Za-z0-9/+=]{40}` (near `aws_secret`) | `wJal...kEXA` |
| Session Token | `aws_session_token\s*=\s*.+` | Variable assignment |
| Account ID | `\d{12}` (in ARN context) | `1234...7890` |

### Google Cloud

| Secret Type | Pattern | Example (Redacted) |
|-------------|---------|-------------------|
| API Key | `AIza[A-Za-z0-9_-]{35}` | `AIza...ZxYw` |
| OAuth Client Secret | `GOCSPX-[A-Za-z0-9_-]{28}` | `GOCS...Ab12` |
| Service Account JSON | `"type":\s*"service_account"` | JSON file detection |
| Firebase Config | `apiKey:\s*["']AIza` | Config object |

### Azure

| Secret Type | Pattern | Example (Redacted) |
|-------------|---------|-------------------|
| Storage Connection | `DefaultEndpointsProtocol=https;AccountName=` | Connection string |
| SAS Token | `SharedAccessSignature=` | URL parameter |
| App Secret | `[a-zA-Z0-9_~.-]{34}` (in Azure context) | Client secret |

## Code Platforms

### GitHub

| Secret Type | Pattern | Length |
|-------------|---------|--------|
| Personal Access Token (classic) | `ghp_[A-Za-z0-9]{36}` | 40 chars |
| OAuth Token | `gho_[A-Za-z0-9]{36}` | 40 chars |
| Server-to-Server | `ghs_[A-Za-z0-9]{36}` | 40 chars |
| Refresh Token | `ghr_[A-Za-z0-9]{36}` | 40 chars |
| Fine-Grained PAT | `github_pat_[A-Za-z0-9]{22}_[A-Za-z0-9]{59}` | 93 chars |

### GitLab
| Secret Type | Pattern |
|-------------|---------|
| Personal Access Token | `glpat-[A-Za-z0-9_-]{20}` |
| Pipeline Token | `glptt-[A-Za-z0-9_-]{20,}` |

## Communication Platforms

### Slack

| Secret Type | Pattern |
|-------------|---------|
| Bot Token | `xoxb-[0-9]{10,}-[0-9]{10,}-[A-Za-z0-9]{24}` |
| User Token | `xoxp-[0-9]{10,}-[0-9]{10,}-[0-9]{10,}-[a-f0-9]{32}` |
| Legacy Token | `xoxs-[0-9]{10,}-[0-9]{10,}-[0-9]{10,}-[a-f0-9]{32}` |
| Webhook URL | `hooks\.slack\.com/services/T[A-Z0-9]+/B[A-Z0-9]+/[A-Za-z0-9]+` |

### Discord
| Secret Type | Pattern |
|-------------|---------|
| Bot Token | `[MN][A-Za-z0-9]{23,}\.[A-Za-z0-9_-]{6}\.[A-Za-z0-9_-]{27,}` |
| Webhook URL | `discord\.com/api/webhooks/\d+/[A-Za-z0-9_-]+` |

## Payment Providers

### Stripe

| Secret Type | Pattern |
|-------------|---------|
| Secret Key | `sk_live_[A-Za-z0-9]{24,}` |
| Publishable Key | `pk_live_[A-Za-z0-9]{24,}` |
| Restricted Key | `rk_live_[A-Za-z0-9]{24,}` |
| Webhook Secret | `whsec_[A-Za-z0-9]{32,}` |
| Test Secret Key | `sk_test_[A-Za-z0-9]{24,}` |

## Private Key Headers

| Key Type | PEM Header |
|----------|-----------|
| RSA | `-----BEGIN RSA PRIVATE KEY-----` |
| EC | `-----BEGIN EC PRIVATE KEY-----` |
| OpenSSH | `-----BEGIN OPENSSH PRIVATE KEY-----` |
| PGP | `-----BEGIN PGP PRIVATE KEY BLOCK-----` |
| DSA | `-----BEGIN DSA PRIVATE KEY-----` |
| PKCS8 | `-----BEGIN PRIVATE KEY-----` |
| Encrypted | `-----BEGIN ENCRYPTED PRIVATE KEY-----` |

## Database Connection Strings

| Database | Pattern |
|----------|---------|
| PostgreSQL | `postgresql://[^:]+:[^@]+@` |
| MySQL | `mysql://[^:]+:[^@]+@` |
| MongoDB | `mongodb(\+srv)?://[^:]+:[^@]+@` |
| Redis | `redis://:[^@]+@` |
| AMQP | `amqp://[^:]+:[^@]+@` |
| SMTP | `smtp://[^:]+:[^@]+@` |

## Common False Positives

| Pattern | Reason |
|---------|--------|
| UUID v4 (`[0-9a-f]{8}-...`) | Structured format, not a secret |
| CSS hex colors (`#[0-9a-fA-F]{6}`) | Predictable format |
| `test_`, `fake_`, `example_` prefixed | Test fixture data |
| Lock file hashes | Package integrity checksums |
| Import paths / module names | Structured identifiers |
| Base64 encoded non-sensitive data | Image data URIs, public certificates |
