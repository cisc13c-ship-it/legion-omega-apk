# LEGIÓN Ω∞

Open-source hybrid Android console + node agent.

**Status: architecture foundation in progress. This branch does not yet contain a production-ready APK.** Remote node control, ADB actions, autonomous skill execution and Aider integration are not implemented until tests and evidence are published.

## Vision
A local-first Android command console and Android node, with separately installed agents for Linux and Windows. The system is built around explicit authorization, inspectable skills, reproducible builds and user-owned infrastructure.

## Planned capabilities
- Android hybrid console and local node.
- Authorized ADB inventory, diagnostics, logs, package metadata and file transfer.
- Linux and Windows node agents.
- CISC policy engine and evidence ledger.
- EDAI skill registry with declarative manifests, pinned dependencies and test gates.
- Aider bridge confined to an isolated Git workspace.
- Offline-first operation and optional self-hosted relay.

## Principles
- Read-only discovery by default.
- No hidden persistence, privilege escalation or credential collection.
- No remote action without node enrollment and scoped authorization.
- AI-generated skills cannot grant permissions to themselves.
- Preserve upstream projects and licenses; do not copy code without license review.
- No capability is considered ready until it passes an observable test.

## Open-source and F-Droid
First-party source is planned under Apache-2.0. Third-party components retain their own licenses and notices. F-Droid is the target distribution channel; inclusion is not yet submitted or approved.

See docs/ARCHITECTURE.md, SECURITY.md, docs/FDROID_RELEASE_PLAN.md, THIRD_PARTY.md and CONTRIBUTING.md.

No release APK is published from this branch. Never install an artifact that is not linked to a source commit and verified build record.
