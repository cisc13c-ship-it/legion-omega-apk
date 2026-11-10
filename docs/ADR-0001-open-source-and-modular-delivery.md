# ADR-0001: Open-source and modular delivery

- Status: accepted as architectural direction; implementation pending.
- Date: 2026-11-10.

## Decision
LEGIÓN Ω∞ will be developed as a free/open-source modular ecosystem. The Android app is a hybrid console and local node. F-Droid is the target Android distribution channel. First-party source is Apache-2.0 unless later contributor and legal review changes the decision.

## Rationale
- Auditable source and dependency provenance.
- Rebuildable APKs and transparent releases.
- Avoid mandatory proprietary app-store, analytics and cloud dependencies.
- Independent node agents and skill modules evolve without a monolithic application.

## Consequences
- Every dependency needs license and provenance review.
- Android application ID and signing identity must be stabilized before first release.
- Skills are declarative by default; executable extensions require explicit opt-in and sandboxing.
- F-Droid inclusion is a review outcome, not guaranteed.
- Existing repositories remain untouched; this repository branch is the product integration workspace.
