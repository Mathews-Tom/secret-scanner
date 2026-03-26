# RULES.md — Secret Scanner

Hard constraints that govern all agent behavior. Non-negotiable.

## ALWAYS

- Redact secrets in output to first 4 and last 4 characters only
- Maintain zero false-negative tolerance — flag uncertain matches as MEDIUM for manual review
- Issue BLOCK verdict when any CRITICAL or HIGH finding exists
- Issue PASS verdict only when zero CRITICAL and zero HIGH findings exist
- Recommend credential rotation for every confirmed secret
- Check .gitignore coverage for flagged files and warn if not ignored
- Run all 6 analysis phases: file selection, provider patterns, private keys, connection strings, env leakage, high-entropy strings
- Exclude binary files, lock files, and .gitignore-matched paths from scanning
- Prioritize high-risk files: config files, .env files, source code, Docker/CI files
- Report findings with provider name, secret type, file:line, and redacted pattern

## NEVER

- Print full secret values in output — the scan report must not become a leak vector
- Skip scanning any file type that could contain secrets
- Override a BLOCK verdict for any reason
- Treat false negatives as acceptable — when in doubt, flag it
- Scan files matching .gitignore patterns
- Report on binary files, lock files, or known-safe generated files

## SHOULD

- Complete scanning in minimal time — this is a pre-commit gate
- Detect environment variable fallbacks that contain real credentials
- Calculate Shannon entropy for strings > 16 characters assigned to credential-named variables
- Exclude known false positives: UUIDs, CSS hex colors, test fixtures with test_/fake_ prefix
- Suggest adding secret-containing file patterns to .gitignore
- Provide git history check command for prior exposure: `git log -p -S '<pattern>'`
