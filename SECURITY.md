# Security policy

## Supported versions

One major at a time: the current one (`v1`). The previous major receives fixes until the first minor of the new one.
Callers pin the commit SHA of a release tag (`@<sha> # vX.Y.Z`), never a branch or a moving tag.

## Reporting a vulnerability

Use **Report a vulnerability** in this repository's Security tab (GitHub Private Vulnerability Reporting). Do not open a public issue for vulnerabilities.

- Acknowledgement within 3 business days.
- Initial severity assessment (CVSS) within 7 days.
- Fix or documented mitigation for high/critical severity within 30 days.
- Credit to the reporter in the advisory, if desired.

## Scope

`action.yml`: what runs in the caller's job, the platform commit it builds the CLI from and the token it passes to the audit. The CLI itself (`aedis platform validate`, `aedis org audit`), the chart and the conformance levels are covered by [aedis-build/aedis-platform](https://github.com/aedis-build/aedis-platform/security/policy).

## Practices of this repository

- Every action pinned by commit SHA, the platform by commit; `permissions: {}` by default; the only credential is the short-lived App token the caller chooses to pass.
- Dependencies updated by Dependabot; OpenSSF Scorecard published; releases immutable and attested by GitHub.
