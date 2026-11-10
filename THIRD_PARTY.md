# Third-party software register

No third-party code is approved for bundling merely because it is useful or open source.

| Candidate | Intended role | License | Decision |
|---|---|---|---|
| AndroidX / Jetpack Compose | Native Android UI | Verify exact artifacts | Candidate |
| ADB protocol libraries such as adb-kt | Authorized ADB transport | Verify exact repository/version | Candidate |
| Dioxamine | ADB/Fastboot/scrcpy reference | Verify repository and all dependencies | Reference only; no code copied |
| Aider | Development assistant on a separate trusted node | Verify upstream license/version | External tool; not embedded |
| Bugjaeger | Android device administration feature reference | Do not assume source/license availability | Behavioral reference only |
| SQLite | Local registry and evidence index | Verify upstream notices | Candidate |

Before adding a dependency, record canonical URL, immutable version/commit, SPDX license, source provenance, transitive dependencies, maintenance/security status, F-Droid compatibility and reason for inclusion.

Do not bundle proprietary SDKs, Google Play Services, Firebase, Crashlytics, advertising or tracking libraries in the F-Droid flavor.
