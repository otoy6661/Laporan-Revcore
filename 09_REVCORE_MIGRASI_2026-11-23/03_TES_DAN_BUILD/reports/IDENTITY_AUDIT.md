# REVCORE REBUILD 161126 — IDENTITY AUDIT (TASK 10)

Date: 2026-11-22
Scope: app source, tests, UI strings, labels, metadata, package names, asset identifiers, ProGuard/inventory references.
Rule applied: internal identities become RevCore; external protocol identifiers, signed asset names, and source attribution are preserved. No blind global replacement: each hit was inspected in context.

## Preserved identity (verified)

| Item | Value | Evidence |
|---|---|---|
| Application ID | `com.revcore.android` | `app/build.gradle.kts:66` |
| Kotlin namespace | `com.revcore.android` | `app/build.gradle.kts:62` |
| App label (ID/EN) | `RevCore` | `app/src/main/res/values/strings.xml:5`, `values-in/strings.xml:5` |
| versionName / versionCode | `1.0` / `1` (unchanged) | `app/build.gradle.kts:69-70` |
| Owner | `otoy` | owner directive; no signing identity or version change made |

## Renamed internal identities

| # | File | Line | Old | Context | Decision | Replacement | Affected files | Re-check |
|---|---|---|---|---|---|---|---|---|
| 1 | `data/maps/RevCoreProfile.kt` | class at 16 | `FlashProProfile` | Internal model for one map profile | Internal identity → RevCore | `RevCoreProfile` | `TuningScreen.kt`, `MapCatalog.kt`, `MapEditor.kt`, `OriginalMapMatcher.kt`, tests | compile + tests |
| 2 | `data/maps/RevCoreProfile.kt` | 65 | `FlashProProfiles::load` TAG | Log tag of the loader companion | Internal identity | `RevCoreProfiles::load` | same | compile |
| 3 | `data/maps/RevCoreDefinitionBuilder.kt` | 10 | `FlashProDefinitionBuilder` | Builder of ordered map sections | Internal identity | `RevCoreDefinitionBuilder` | `MapEditor.kt`, tests | compile + tests |
| 4 | `data/maps/MapDefinition.kt` | 4 | `FlashProViewType` | Map view type enum | Internal identity | `RevCoreViewType` | parser/editor/tests | compile + tests |
| 5 | `test/.../TestRevCoreProfiles.kt` | 11 | `TestFlashProProfiles` | Test fixture loader | Internal identity | `TestRevCoreProfiles` | 3 test classes | tests |
| 6 | `core/comm/LiveDataProfile.kt` | 350, 373, 393, 414, 425, 466, 487 | `flashpro.dat17` … `flashpro.datD1` | Live-table profile ids | Internal identity | `revcore.dat17` … `revcore.datD1` | engine lookup, tests | tests |
| 7 | `core/comm/HondaEcuProfiles.kt` | 209-271 | labels `(FLASHPRO dat17)` … | User-visible live-table labels | Internal label → RevCore; firmware attribution remains in the KDoc above and in the profile notes | `(revcore.dat17)` … | Live Data label rendering | compile |
| 8 | Package `data/flashpro` | 23 main + 9 test files | `com.revcore.android.data.flashpro` | Internal source package | Neutral functional identity; avoids third-party name in code namespace | `com.revcore.android.data.maps` | all imports, `app/proguard-rules.pro:31` | full build; APK dex contains `data/maps`, no `data/flashpro` |
| 9 | `ui/screens/TuningScreen.kt` | several | "Metadata profil FlashPro", "source VomConfig #", "Database MAP asli FlashPro", "Single FlashPro-style Tuning system" | User-facing strings/comments | Internal labels → RevCore; VomConfig replaced by "profil #" because the UI must not expose the research schema as an identity | "Metadata profil RevCore", "sumber profil #", "Database MAP RevCore — arsip referensi", "Single RevCore Tuning system" | Tuning UI | compile |
| 10 | `data/maps/MapParser.kt` | 225 | `FlashPro hanya memakai rows` | Validation message shown to the user | Internal rule label | `RevCore hanya memakai rows` | Tuning validation UI | compile/tests |
| 11 | `data/maps/MapValidator.kt` | 96 | `kelas FlashPro` | Validation message shown to the user | Internal image class | `kelas image RevCore` | Tuning validation UI | compile/tests |
| 12 | `data/maps/RevCoreProfile.kt` | 10-15 | "One FlashPro `VomConfig` record" | KDoc | Keep the provenance but name the RevCore type | "One RevCore map-profile record, reconstructed from the FlashPro `VomConfig` schema" | KDoc only | compile |
| 13 | `data/maps/RemapWorkflow.kt` | — | "Container FlashPro tanpa magic" | Error message | Internal container terminology | "Container MAP tanpa magic" | Tuning write flow | compile/tests |
| 14 | `ui/tuning/TuningState.kt` | — | "profil FlashPro" (ambiguous message), KDoc "FlashPro" | Internal state message/docs | Internal identity | "profil RevCore"; attribution kept in KDoc | Tuning state | `TuningStateTest` |
| 15 | `data/maps/OriginalMapMatcher.kt` | — | UI-facing "VomConfig #" notes | User note in map matching result | Internal label | "profil #" | Tuning open flow | compile |

## Preserved external identifiers (deliberately NOT renamed)

| Item | Where | Why retained |
|---|---|---|
| `flashpro_profiles.json` | `assets/`, `RevCoreProfile.kt:138-145`, `TuningScreen.kt:1341`, `app/build.gradle.kts:53`, `security/GateEvaluator.kt:82` | Signed/verified asset name and integrity inventory entry. Renaming would break the Gate 10 inventory and the owner's asset archive. |
| `flashpro_maps/` and `flashpro_maps/manifest.tsv` | `BundledMapManifest.kt:46-48`, `app/build.gradle.kts:54`, `GateEvaluator.kt:83`, tests | External archive layout of the owner-provided MAP set; treated as data, not app identity. |
| `docs/no4_verification/flashpro_profiles.json` | research archive path | Research source referenced by the order; must not be deleted or renamed. |
| `VomConfig` / `BinLoader` / `flashproDomain.*` | KDoc in `RevCoreProfile`, `MapCodec`, `MapCrypto`, `MapParser`, `RevCoreDefinitionBuilder` | Attribution of the decompiled source schema and codec behavior; required provenance, not branding. |
| "proven FLASHPRO probe order" | KDoc `HondaEcuProfiles.kt:201`, `HondaProtocolEngine.kt:1309` | Factual evidence of how the table order was obtained. |
| "HDiag" | `core/comm/LivePollPolicy.kt:7` ("it is not derived from HDiag") | Single factual statement; no H-Diag identity remains in code, labels, or metadata. |
| "Honda" in UI descriptions | e.g. Diagnostics subtitle "Honda ECM Diagnostic Scanner" | Target vehicle/protocol identity; required for user context, not a third-party product identity. |
| `FTDI`, `FT232R`, `K-Line`, `Keihin`, `Shindengen` | drivers, remap engines, UI evidence text | Required hardware/protocol identifiers; cannot be renamed without losing technical meaning. |
| `source` metadata inside asset JSON | `assets/flashpro_profiles.json` | Documents provenance of the map metadata; kept unchanged. |

## Checks performed

- `grep -rn "FlashPro|FLASHPRO|flashpro"` over `app/src/main` and `app/src/test`; every remaining hit is listed above as an asset name, research path, or attribution comment.
- `grep -rn "H-Diag|HDiag|Hdiag"` → one KDoc statement only.
- `grep -rn "data\.flashpro"` over source → no hits after rename.
- Debug APK `classes*.dex` inspected: contains `com/revcore/android/data/maps`, contains no `com/revcore/android/data/flashpro`.
- `proguard-rules.pro` keep rule updated to `com.revcore.android.data.maps.**`.
- Full build and 435 unit tests pass after the rename.

## Not changed (explicit)

- `LegacyHondaProtocolEngine` remains untouched (unused legacy class; "Honda" is the target protocol/vehicle name, not FlashPro branding).
- No asset was renamed, deleted, re-signed, or replaced.
- No version, applicationId, owner identity, or signing configuration was changed.
