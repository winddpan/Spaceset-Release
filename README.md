# Spaceset releases

`Spaceset/Info.plist` defines the update feed and Sparkle public key. The packaged
App's version, numeric build and signed beta cutoff determine release metadata.
Spaceset uses Tangrid's existing Sparkle keychain key; private keys are never exported.

Build with Xcode, `create-dmg`, `jq`, `rg`, a Developer ID certificate and the
existing `TangridNotary` notarytool keychain profile:

```sh
./build_spaceset.sh
```

This archives the Spaceset scheme, signs and notarizes the App and DMG, staples
their tickets, creates the signed Sparkle ZIP, and merges its appcast item using
Tangrid's existing appcast generator. Outputs are kept here; archives and dSYMs
are retained in `build/Spaceset/`. Nothing is published during packaging.

Each beta expires 14 days after packaging starts. The fixed UTC cutoff is embedded
in the signed App. Override `BETA_VALID_DAYS` or supply `BETA_EXPIRATION_DATE`:

```sh
VERSION=0.1.1-beta ./build_spaceset.sh
BETA_EXPIRATION_DATE=2026-11-17T08:00:00Z ./build_spaceset.sh
```

Direct Xcode builds use `SPACESET_BETA_EXPIRATION_DATE` from the Spaceset target
settings. Stable versions ignore the beta cutoff. A beta with missing or invalid
cutoff metadata is locked. Expiration is checked before onboarding or business
startup, throughout runtime, on activation and after wake. Closing the required
Sparkle update session exits the expired App, including offline/no-update cases.

Update the appropriate English and Chinese changelogs before packaging. Each
release needs a unique marketing version/tag. `BUILD_NUMBER` defaults to a Unix
timestamp and must increase for each published build.

Publish explicitly with an authenticated GitHub CLI:

```sh
./release_spaceset.sh 0.1.0-beta
```

The publisher validates the packaged App, notarization tickets and Sparkle ZIP
signature, bootstraps an empty `winddpan/Spaceset-Release` repository on `main`,
uploads ZIP/DMG to a draft release, publishes it, then updates appcast and changelogs.
Beta releases are marked as prereleases. The live feed is the source of truth for
the other channel and existing critical-update thresholds. Git push never forces.
If a remote publish fails, inspect the draft/release before retrying; the script
stops on existing tags/releases rather than overwriting assets.
