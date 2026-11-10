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


## Connection Hub and agent federation

The Connection Hub is the single policy-controlled registry for external accounts, local devices, remote nodes and assistant providers. It centralizes identity and consent without centralizing raw credentials or granting every agent universal access.

### Core design
- **Connection Registry:** records provider, account alias, owner, connection state, approved scopes/capabilities, token location, expiry/health metadata and revocation state. It must not store raw secrets in ordinary records.
- **Policy Decision Point (PDP):** evaluates every requested action against actor, resource, operation, scope, risk tier, session and user approval.
- **Policy Enforcement Point (PEP):** sits at every connector and node boundary. A tool is unavailable unless the PEP authorizes the exact operation.
- **Credential Vault:** stores OAuth refresh tokens and node private keys separately from application data, encrypted at rest and accessible only to the connector process that needs them.
- **Agent Gateway:** exposes typed, allowlisted tools to ChatGPT, Gemini, Grok, Ollama and other providers. Providers receive short-lived, task-scoped capabilities, never reusable provider credentials or raw OAuth tokens.
- **Evidence Ledger:** records authorization decisions and operation outcomes, with sensitive payloads redacted by default.

### Capability grants
A grant is bound to one principal, one connection/resource set, one operation family and an expiry. The baseline capability vocabulary is:
- DISCOVER: list available connections and non-sensitive metadata.
- READ_METADATA: search/list metadata without opening content.
- READ_CONTENT: read only selected or explicitly scoped content.
- WRITE: create or modify specified resources.
- SEND/EXECUTE: perform externally visible actions, such as sending email or running an approved task.
- DELETE: remove specified resources; always a separate high-risk grant.
- ADMIN: change policies, enroll/revoke nodes or manage credentials; never inherited by an AI agent.

Grants are deny-by-default, revocable, time-bounded and non-transitive. An agent cannot delegate a capability it does not possess. A connection being present does not imply that any agent may use it.

### Request and approval lifecycle
1. An agent proposes a typed action with purpose, target, data minimization plan and requested capability.
2. The PDP evaluates policy and risk. READ operations may be auto-approved only when the user has configured a matching narrow policy.
3. WRITE, SEND, DELETE, EXECUTE and ADMIN require a separate approval UX showing the exact target and effect, unless the user has explicitly configured a narrowly bounded automation rule.
4. The PEP executes the exact approved operation; parameters cannot be silently expanded after approval.
5. The Evidence Ledger records actor, connection alias, resource reference, capability, policy decision, approval reference, timestamp, result and correlation ID.
6. Verification checks the postcondition. Failure triggers a safe stop and, where supported, a documented compensating action; irreversible operations are never described as rollbackable.

### Provider-neutral connector contract
Every connector implements a small typed contract: discover capabilities, authorize, inspect connection health, execute an allowlisted operation, revoke, and report evidence. Provider-specific APIs remain behind adapters. A connector must declare supported operations, required scopes, data classes, rate limits, retention behavior and tests before it can be enabled.

### Data boundaries and privacy
- Minimize content returned to an agent; prefer metadata, excerpts and user-selected resources over mailbox-wide or drive-wide ingestion.
- Do not train, index or retain external content by default. Any cache/index must have an explicit purpose, retention period, deletion path and source reference.
- Treat files, emails, webpages and tool outputs as untrusted data, never as instructions that can override system policy.
- Separate user data, connector secrets, agent context and audit evidence into distinct storage domains.
- Redact secrets and message bodies from ordinary logs. Audit records should identify the resource and action without duplicating the resource content.
- Disconnecting a provider revokes its credentials where supported, disables dependent tools and marks cached data for deletion according to the retention policy.

### Node and agent trust
- Nodes enroll with an explicit user action and a cryptographic identity. Use mutual authentication, replay protection, short-lived sessions and per-node revocation.
- A node's advertised capabilities are descriptive, not permission. The policy engine independently authorizes each operation.
- Keep Android, Linux and Windows agents least-privileged and platform-specific. The control plane must not expose arbitrary remote shell as a generic LLM tool.
- Assistant providers are untrusted requestors, not administrators. They can propose plans and consume scoped tool results; they cannot change policy, grant permissions, access vault internals or enroll nodes.

### Initial implementation order
1. Implement core-model contracts for Connection, Principal, CapabilityGrant, Approval, PolicyDecision and EvidenceEvent.
2. Implement local SQLite metadata plus append-only JSONL evidence; keep secrets out of both.
3. Implement policy evaluation and a fake connector with deterministic tests before connecting real accounts.
4. Add Google Drive metadata and selected-file READ through the existing OAuth plan.
5. Add Gmail metadata/search under a separate grant; defer send and delete.
6. Add the local agent gateway and one allowlisted MCP tool; verify denial paths as thoroughly as success paths.
7. Add LAN node enrollment only after the local policy and evidence tests pass.

### Acceptance gates
- Missing grant, expired grant, wrong principal, wrong resource, scope escalation and replayed request are denied.
- Changing an operation's target or parameters after approval invalidates that approval.
- Revocation blocks the next operation and disables dependent agent tools.
- Logs and evidence contain no access token, refresh token, password or private key.
- A connector cannot invoke an operation outside its declared allowlist.
- Every successful mutation has a verifiable postcondition or is clearly marked unverified.
- Disconnect and cache deletion behavior are covered by tests.

