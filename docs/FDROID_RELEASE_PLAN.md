# F-Droid release plan

## Target
Publish the first-party Android application from public source, built from source by F-Droid, without proprietary tracking or advertising SDKs.

## Required before submission
- Real, functional source code in the public upstream repository.
- FOSS license for the complete app and every bundled dependency.
- Buildable command-line toolchain and pinned dependency versions.
- Unique, stable Android application ID.
- No Google Play Services, Firebase, Crashlytics, proprietary analytics or ad SDKs in the F-Droid flavor.
- No undisclosed network behavior or anti-features.
- App metadata, screenshots, icon, changelog, source URL and issue tracker.
- Tagged source commit for each release.
- Build recipe in fdroiddata and a clean F-Droid build.
- Security review, dependency inventory and software bill of materials.

## Reproducibility
Reproducible builds are a best practice and a release goal from the beginning, even though F-Droid's quick-start guide says they are not a strict inclusion requirement. Pin toolchain versions, remove timestamps and nondeterministic inputs, use command-line Gradle builds and compare upstream and F-Droid artifacts.

## Distribution stages
1. Public source and developer-signed test artifacts.
2. Self-hosted F-Droid-compatible test repository for opt-in testers.
3. Reproducible release candidate and source tag.
4. Submit metadata to the official fdroiddata repository.
5. Official listing only after review and acceptance.

## Dynamic skills and executable modules
F-Droid policy requires explicit opt-in before an app downloads additional executable binaries, with clear disclosure that the user is bypassing F-Droid's checks. Default skill catalog therefore uses declarative manifests and built-in operations. Executable extensions are separately disclosed, opt-in, integrity-checked and disabled by default.

## Signing
Keep signing keys offline or in a dedicated signing environment. Never commit keys or passwords. Decide the long-term signing model before the first release because Android update compatibility depends on signing identity.
