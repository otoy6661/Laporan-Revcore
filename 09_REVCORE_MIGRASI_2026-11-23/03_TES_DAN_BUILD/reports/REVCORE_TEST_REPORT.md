# REVCORE TEST REPORT (Phase C)

Date: 2026-11-15. Boundary: source + JVM unit tests only. No APK built, signed or installed.

## 1. Commands executed

| Command | Result |
|---|---|
| `gradle.bat :app:testDebugUnitTest --offline --console=plain` | **BUILD SUCCESSFUL** |
| `gradle.bat :app:compileDebugKotlin --offline` | **BUILD SUCCESSFUL** |
| `gradle.bat :app:compileReleaseKotlin --offline` | **BUILD SUCCESSFUL** |

Launcher used because this project has no Gradle wrapper:
`C:\Users\Lord Otoy\.gradle\wrapper\dists\gradle-9.6.0-bin\42k10rwplmzkhuboz9kdazi7s\gradle-9.6.0\bin\gradle.bat`.

## 2. Result

`tests=295 skipped=0 failures=0 errors=0` — 34 classes.

Delta this phase: **+25** tests (270 → 295).

| Class | Tests | Note |
|---|---:|---|
| `security.IntegrityGateTest` | 31 | existing gate/record/manifest suite, extended earlier |
| `security.GateStagingTest` | **13** | new: order, per-gate decisions, prerequisites, Gate 8 |
| `security.InstallationMarkerTest` | **10** | new: marker container, MAC, schema, tamper |
| `security.TrustRecordTest` | 11 (+2) | new: format 2 and legacy rejection |
| `security.UiIntegrityManifestTest` | 13 | unchanged |
| `security.ApkArchiveTest` | 11 | unchanged |
| `security.UiManifestDigestParityTest` | 3 | unchanged |
| `security.BinaryXmlStructureParityTest` | 3 | unchanged |
| `security.SecurityGsonModelTest` | 2 | now includes `InstallationMarkerPayload` |
| other packages (core/data/ui) | 198 | unchanged, still green |

## 3. What the new tests prove

### `GateStagingTest` (13)

- `stagedOrderCoversExactlyTheTenGatesOnce` — the executed constant contains all ten gates,
  none twice.
- `finalTrustRunsLastAndTheEvidenceGatesRunOneToTen` — order is `1,2,3,4,5,6,7,9,10,8`;
  `FINAL_TRUST.number == 8`.
- `finalTrustAloneIsNeverChecked` — Gate 8 alone is `NOT_CHECKED` / `SKIPPED`.
- `everyEvidenceGatePassesOnFullyVerifiedFacts` — all nine pass on consistent facts.
- `finalTrustPassesOnlyWhenAllNineGatesPassed` — Gate 8 PASS only with nine PASS.
- `anyFailedGateFailsFinalTrust`, `aBypassedGateBlocksFinalTrustInAReleaseBuild`,
  `anUncheckedGateBlocksFinalTrustInAReleaseBuild`, `anEmptyResultListFailsClosed` — every
  non-PASS state blocks Gate 8.
- `invalidUiSignatureFailsItsConsumersButNotApkIdentity` / 
  `invalidPolicySignatureFailsEveryPolicyConsumer` — prerequisite downgrade, independently of
  the APK identity gate.
- `debugBuildWithoutPolicyBypassesEveryGateIncludingFinalTrust` — BYPASS ≠ PASS.
- `statusToPhaseNeverTurnsABypassOrASkipIntoAPass` — UI mapping cannot fabricate a pass.

### `InstallationMarkerTest` (10)

- Seal/open round trip with the exact header/length/MAC layout.
- A different MAC key is rejected.
- **Every single-byte modification is rejected** (all offsets, header and payload).
- Truncation, extension, undersize and oversize files are rejected.
- Wrong magic and wrong container version are rejected.
- Authenticated but schema-invalid payloads are rejected: bad `installId`, blank package,
  zero versionCode, blank versionName, short/non-hex digests, `recordFormat = 1`,
  `format = 2`.
- Oversized payload cannot be sealed.
- File bounds documented and asserted (8 / 32 / 3072 / 3112).
- 1000 generated `installId`s are 32-hex and all distinct.
- `installIdDigest` is deterministic and content-sensitive.

### `TrustRecordTest` additions (2)

- `format == 2`, JSON carries `installId` and `installationStateDigest`, round trip preserves
  equality.
- A legacy `{"format":1}` document is not the current format and has no `installId`.

### `SecurityGsonModelTest`

- `InstallationMarkerPayload` added to the reflection check, so a future field cannot silently
  rely on a name R8 may rename.

## 4. What is deliberately not asserted (would require `Context` / device)

- `InstallationStateStore.load/save/deleteAndVerify` on real storage, atomic rename.
- Keystore HMAC alias creation and key invalidation.
- `pairMatches` / `verifyStartupRecord` with a real APK and `PackageManager`.
- `evaluateStartup(forceFull=…)` against a real record file.
- Commit ordering, commit failure, double LANJUTKAN, rotation, process death.
- Any Compose/instrumented UI assertion.

## 5. Unverified / BLOCKED scenarios

1. Visible examination on device: ten rows, labels, Gate 8 last, tags.
2. Dialog cannot be dismissed by back/outside; `LANJUTKAN` commits once.
3. Failure screen exact strings; `PERIKSA ULANG` runs a real FULL.
4. Rotation during examination keeps the job; new process does not show the dashboard first.
5. Clear Data, update, force stop, swipe-away behaviour.
6. APK mutation matrix rerun (resource, icon, label, credits, XML, `resources.arsc`, missing,
   extra, duplicate, DEX-unchanged with changed resource, invalid manifest signature).
7. MT Manager repack scenarios.
8. Physical ECU / K-Line / Live Data regression on hardware.
9. FULL vs TRUST_RECORD timing.

These are reported as not tested, not as passing. They require an authorized build and a
device/emulator run.
