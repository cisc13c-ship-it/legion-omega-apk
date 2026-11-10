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
