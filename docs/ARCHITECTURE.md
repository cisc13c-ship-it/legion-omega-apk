# LEGIÓN Ω∞ — Open modular architecture

## Product shape
One Android app with two roles: a console for enrolling and administering authorized nodes, and a local Android node that reports capabilities and performs only locally authorized operations. Linux and Windows use separately installed agents.

## Planes
- Control: node registry, enrollment, policy, task plans and approvals.
- Data: authenticated node channels and bounded task payloads.
- Evidence: append-only event ledger, artifact hashes, build provenance and rollback references.
- Extension: declarative skill manifests and separately sandboxed executors.

## Modules
- app-android: Kotlin + Jetpack Compose console and Android node.
- core-model: typed node, capability, task and skill contracts.
- core-policy: deny-by-default authorization and approval state machine.
- core-evidence: SQLite index, append-only JSONL events and SHA-256.
- adapter-adb: authorized ADB operations behind a narrow interface.
- node-linux and node-windows: minimal OS-specific agents.
- node-android: optional Termux-compatible companion agent.
- link: LAN transport first; optional self-hosted relay later.
- aider-bridge: Aider on a development node, confined to a Git worktree.
- edai-skill-registry: declarative manifests, pinned dependencies, tests and lifecycle.

## ADB strategy
Use Android-supported ADB pairing and authorization. First release: read-only inventory and diagnostics. Package changes, shell execution, file writes and reboot operations are separate permission-gated capabilities. Fastboot, bootloader and flashing are excluded from the initial release.

Bugjaeger is a functional reference, not a source dependency. Dioxamine and adb-kt are candidates for technical and license review; no code is copied until exact repository, license, version and transitive dependencies are verified.

## Aider strategy
Aider runs on a trusted development node, not as an embedded privileged APK process. The bridge accepts bounded tasks against a dedicated Git worktree, returns diffs and test logs, and cannot issue remote device commands. Merge, deployment and skill activation remain separate approval steps.

## Skill model
A skill is a versioned declarative package, not unrestricted model-generated shell text. It includes ID, semantic version, platform, typed inputs/outputs, least-privilege permissions, pinned dependencies, integrity hash, provenance, tests, sandbox profile and lifecycle state.

Default format is declarative JSON/YAML with a restricted operation vocabulary. Executable extensions are opt-in, disclosed, integrity-checked and sandboxed. No silent download or auto-execution.

## Node link
Phase 1: local LAN transport, mutual enrollment and explicit approval.
Phase 2: optional self-hosted relay, outbound-only agent connection, short-lived credentials and reconnect backoff. No public relay or mandatory cloud dependency.

## Autonomous configuration loop
DISCOVER -> CLASSIFY -> PLAN -> CHECKPOINT -> APPROVE -> APPLY -> VERIFY -> ROLLBACK/RECORD.

Discovery, planning, manifest validation and sandbox tests may run automatically. Node changes, package installation, scripts, file writes, restarts and administrative operations require explicit scoped authorization.

## Non-goals for v0
Rooting devices, bypassing Android prompts, hidden persistence, credential extraction, arbitrary remote shell exposed to an LLM, self-modifying production binaries or silent executable skill downloads.


## Google account integration — OAuth connector

Google account linking is an optional integration, not a prerequisite for the core app. It uses Google's official OAuth 2.0 authorization and APIs; LEGIÓN never asks for or stores the user's Google password.

### Initial services
- Google Drive: list/search metadata and read selected files first; upload/create/update only after a separate WRITE grant.
- Gmail: begin with message search and metadata; read message bodies only with a separately explained scope; sending mail is a distinct EXECUTE-like capability requiring explicit confirmation.
- Later candidates: Calendar and Contacts, each independently enabled and scoped.

### Least-privilege model
- Use incremental OAuth consent: request a scope only when the user enables the corresponding feature.
- Prefer narrow Drive scopes such as app-created files or user-selected files where those fit; do not default to full Drive access.
- Separate READ, WRITE, SEND and DELETE capabilities. Never bundle them into one broad "Google access" toggle.
- Show the exact account, granted scopes, last sync, connected nodes and data-retention policy.
- Support disconnect, token revocation and local deletion of cached data.
- Store refresh tokens only in Android Keystore-backed encrypted storage or a dedicated self-hosted secret store; never in Git, logs, skill manifests or plain SQLite.
- Keep tokens on the user's device or trusted self-hosted gateway. Do not forward raw OAuth tokens to language models or arbitrary nodes.
- Audit every API operation with actor, scope, resource ID, action, result and timestamp; redact message contents and secrets from logs.

### ChatGPT / MCP bridge
An optional CISC-MCP Google connector can expose a small allowlisted tool surface (for example: search Drive files, read a user-selected file, search Gmail metadata). The bridge must authenticate independently, enforce the same policy engine and return only the minimum requested content.

Linking Google to LEGIÓN does **not** automatically grant ChatGPT access. For ChatGPT to use the integration, the MCP service must be reachable through a supported connection method and separately connected/authorized by the user in ChatGPT. If that connection is unavailable, LEGIÓN can still use Google APIs locally and export a selected result for the user to share.

### Google policy and F-Droid constraints
Google OAuth scopes may be sensitive or restricted. Public distribution may require OAuth app verification, privacy disclosures, domain ownership and, for restricted-scope server-side access, an independent security assessment. Keep separate development and production OAuth projects. Confirm current scope classification before implementation.

F-Droid compatibility must be evaluated for the chosen OAuth flow and dependencies. No proprietary Google Play Services SDK is required as an architectural assumption; prefer standards-based OAuth and HTTPS APIs where practical. Google APIs remain a third-party online service and the app must disclose that this optional feature contacts Google.

### Safe rollout
1. Local prototype with a test Google Cloud project and one test account.
2. Drive metadata and selected-file READ only.
3. Gmail metadata search, then selected message reading behind separate consent.
4. Write/upload and email send only after independent review and confirmation UX.
5. MCP bridge only after local OAuth, token storage and policy enforcement are tested.
