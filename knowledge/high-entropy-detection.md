# High-Entropy String Detection Reference

Methods for identifying potential secrets via information entropy analysis.

## Shannon Entropy

### Formula
```
H = -sum(p(x) * log2(p(x))) for each character x in string
```

Where `p(x)` = frequency of character `x` / total string length.

### Entropy Thresholds

| Character Set | Typical Secret Entropy | Threshold for Flagging |
|---------------|----------------------|----------------------|
| Hex (0-9, a-f) | 3.5 - 4.0 bits/char | > 3.5 |
| Base64 (A-Z, a-z, 0-9, +, /) | 4.5 - 6.0 bits/char | > 4.5 |
| Alphanumeric (A-Z, a-z, 0-9) | 4.0 - 5.5 bits/char | > 4.5 |
| Full ASCII printable | 4.5 - 6.5 bits/char | > 5.0 |

### Reference Entropy Values

| String Type | Typical Entropy (bits/char) | Flag? |
|-------------|---------------------------|-------|
| English word | 2.0 - 3.5 | No |
| UUID v4 | 3.7 | No |
| Base64 secret | 5.0 - 6.0 | Yes |
| Random hex (API key) | 3.8 - 4.0 | Context-dependent |
| AWS secret key (base64) | 5.2 - 5.8 | Yes |
| Password hash (bcrypt) | 4.5 - 5.0 | No (known format) |

## Context Signals

### Variable Name Indicators (flag when entropy is high)
- `key`, `api_key`, `apiKey`, `API_KEY`
- `secret`, `secret_key`, `secretKey`
- `token`, `access_token`, `auth_token`
- `password`, `passwd`, `pwd`
- `credential`, `credentials`
- `connection_string`, `conn_str`
- `private_key`, `signing_key`

### Context Indicators (increase suspicion)
- Assignment in configuration blocks
- Near `os.environ.get()` with default value
- Near `process.env.` with fallback
- Inside `.env`, `.yaml`, `.toml`, `.ini`, `.cfg` files
- In environment variable setup scripts

## Minimum String Length

| Check Type | Min Length | Rationale |
|-----------|-----------|-----------|
| Entropy analysis | 16 chars | Shorter strings produce unreliable entropy |
| Hex pattern | 32 chars | 128-bit minimum for meaningful secret |
| Base64 pattern | 20 chars | Covers most API key lengths |
| General scan | 8 chars | Catch short passwords in config |

## False Positive Reduction

### Exclusion Rules
1. Strings matching UUID format: `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}`
2. Strings that are valid URLs without credentials
3. Strings in test files with `test_`, `mock_`, `fake_` context
4. Hash outputs in known positions (git SHAs, integrity hashes)
5. CSS/HTML color codes
6. Encoded but non-secret data (base64 images, SVG data URIs)
7. Strings inside comments referencing documentation

### Encoding Detection

| Encoding | Pattern | Entropy Impact |
|----------|---------|---------------|
| Base64 | `[A-Za-z0-9+/]+=*$` | Higher entropy due to wider charset |
| Hex | `[0-9a-fA-F]+$` | Lower entropy due to 16-char alphabet |
| Base32 | `[A-Z2-7]+=*$` | Moderate entropy |
| URL-safe Base64 | `[A-Za-z0-9_-]+$` | Similar to standard Base64 |

## Implementation Notes

- Calculate entropy per-string, not per-file
- Apply variable name context before entropy threshold
- A string with entropy > 4.5 in a variable named `key` = HIGH finding
- A string with entropy > 4.5 in a variable named `message` = MEDIUM (manual review)
- Strings with entropy > 5.5 in any context = flag regardless of variable name
