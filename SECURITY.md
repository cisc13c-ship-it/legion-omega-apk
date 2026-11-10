# Security Policy

Only tagged releases are supported. Development branches are not production releases.

## Reporting
Do not publish exploit details, credentials, private keys, device identifiers or personal data in public issues. Request a coordinated disclosure path through the repository maintainers.

## Security invariants
- Node enrollment is explicit and mutual.
- READ is the default permission.
- WRITE, EXECUTE and ADMIN are separate capabilities and are never inherited implicitly.
- AI-generated skills cannot grant permissions to themselves.
- Every node command is authorized at execution time.
- Secrets are stored outside Git and excluded from logs.
- No inbound administration port is exposed by default.
- Destructive operations require a checkpoint where feasible, explicit confirmation and a recovery path.
- Remote executable skill installation is opt-in and integrity-checked.
- Security failures fail closed and are recorded.

## Release gates
Threat model, dependency/license scan, static analysis, unit and integration tests, APK manifest review, permission review, secret scan and build provenance are required before release.
