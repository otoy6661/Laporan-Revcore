# RevCore — Test & Verification Report

Build: `RevCore build 151126.apk` (hardened rebuild, 15 November 2026 13:41 WIB)
Date: 15 November 2026
Package: `com.revcore.android`
Version: 1.0 (versionCode 1)
Size: 39,623,151 bytes
SHA-256: `0072f4cbf8907ceb0c48f8b98ba3cb249cc570808c8beeee05922455e53c89aa`

> Revision note: sections 1–10 describe the test run for the earlier `151126`
> artifact (`cd8ef5cb…09177d`, now archived). Section 11 records the hardened
> rebuild and supersedes the package-level results. See
> `RevCore HARDENING VERIFICATION 151126.md`.

Verification labels used below:

- **source-code verified** — inspected in the current source tree.
- **automated test verified** — executed by a test run recorded in this report.
- **APK-package verified** — checked against the exported APK bytes with
  `apksigner`, `aapt2`, an apksig-based policy verifier, or archive inspection.
- **hardware verified** — executed on a physical adapter + ECU.
- **not yet verified** — no evidence was produced in this session.

## 1. Summary

| Area | Result | Evidence label |
|------|--------|----------------|
| JOB 1 — AFR honesty | Passed | source-code + automated test + APK-package |
| JOB 2 — ten gates | Passed at implementation/test/package level | source-code + automated test + APK-package |
| JOB 3 — philosophy section | Passed | source-code + APK-package (strings present) |
| Regression suite | 214 / 214 passed | automated test |
| Release signature and identity | Passed | APK-package |
| Signed integrity policy in APK | Passed | APK-package (RSA signature verified) |
| Hardware behaviour (USB / K-Line / ECU) | Not run | not yet verified |
| On-device UI interaction | Not run | not yet verified |

Failed checks: none in the executed scope.

## 2. Automated tests

Command:

```
JAVA_HOME="C:/Program Files/Android/Android Studio/jbr" \
  ~/.gradle/wrapper/dists/gradle-9.6.0-bin/42k10rwplmzkhuboz9kdazi7s/gradle-9.6.0/bin/gradle \
  :app:testDebugUnitTest --offline
```

Environment: Gradle 9.6.0, Android Gradle Plugin 9.1.1, Android Studio JBR
(Java 25.0.3), compileSdk 36, minSdk 26. Results were read from
`app/build/test-results/testDebugUnitTest` (26 suites).

| Suite | Tests | Failures | Errors | Skipped |
|-------|------:|---------:|-------:|--------:|
| core.comm.EcuProfilesTest | 9 | 0 | 0 | 0 |
| core.comm.EcuSessionManagerTest | 8 | 0 | 0 | 0 |
| core.comm.HondaFrameCodecTest | 10 | 0 | 0 | 0 |
| core.comm.KlineWireEngineTest | 8 | 0 | 0 | 0 |
| core.comm.LiveDataProfileParserTest | 21 | 0 | 0 | 0 |
| core.comm.LivePollPolicyTest | 2 | 0 | 0 | 0 |
| core.protocol.BaselineProtocolTest | 6 | 0 | 0 | 0 |
| core.protocol.HondaIdentifierParserTest | 7 | 0 | 0 | 0 |
| core.protocol.KlineTransceiverTest | 14 | 0 | 0 | 0 |
| core.protocol.SingleFlightGateTest | 3 | 0 | 0 | 0 |
| core.tuning.TuningLogStoreTest | 10 | 0 | 0 | 0 |
| core.usb.AdapterConnectionPolicyTest | 12 | 0 | 0 | 0 |
| data.ecu.EcmCatalogAndIdentityTest | 4 | 0 | 0 | 0 |
| data.ecu.EcmIdentityMatcherTest | 9 | 0 | 0 | 0 |
| data.ecu.EcmIdentityTest | 13 | 0 | 0 | 0 |
| data.ecu.EcmIndonesianReferenceTest | 4 | 0 | 0 | 0 |
| data.ecu.EcuIdCanonicalizerTest | 14 | 0 | 0 | 0 |
| data.flashpro.BundledMapAssetsTest | 4 | 0 | 0 | 0 |
| data.flashpro.MapCatalogFlashEngineTest | 4 | 0 | 0 | 0 |
| data.flashpro.MapCodecTest | 6 | 0 | 0 | 0 |
| data.flashpro.MapCryptoTest | 7 | 0 | 0 | 0 |
| data.flashpro.MapParserEditorTest | 10 | 0 | 0 | 0 |
| data.flashpro.OriginalMapArchiveAnalysisTest | 1 | 0 | 0 | 0 |
| data.flashpro.RemapWorkflowTest | 4 | 0 | 0 | 0 |
| security.IntegrityGateTest | 16 | 0 | 0 | 0 |
| ui.tuning.TuningStateTest | 8 | 0 | 0 | 0 |
| **Total** | **214** | **0** | **0** | **0** |

## 3. JOB 1 — AFR evidence

- `LiveDataProfileParserTest.dat20_mapsO2AndStfOnlyAndNeverFabricatesAfr` —
  dat20 exposes narrowband O2 voltage and STF only; no AFR key is produced.
- `LiveDataProfileParserTest.afrIsReportedUnsupportedWithoutOverridingParsedValues`
  — merging an O2-only table marks AFR `UNSUPPORTED`; a parsed AFR would win,
  never be overwritten by the fallback.
- `LiveDataProfileParserTest.afrHasNoFieldInAnyReconstructedTable` — no
  reconstructed Honda table declares an AFR sensor.
- The compiled APK contains both notices: `AFR TIDAK DIDUKUNG` and
  `AFR NOT SUPPORTED` (one occurrence each, `classes.dex`).
- No RPM, TPS, temperature, or narrowband-voltage derivation was added.
- K-Line frames, baud rate, and table timing were not changed by JOB 1.

Label: source-code verified, automated test verified, APK-package verified.
Hardware confirmation that a Beat K1A ECU never publishes AFR: not tested on a
bike in this session.

## 4. JOB 2 — ten gates and signed policy

All 16 `IntegrityGateTest` cases passed, including:

- `releasedPolicyWithMatchingEvidencePassesAllTenGates`
- `tamperedDatabaseFailsResourceAndDatabaseGates`
- `invalidDatabaseSchemaFailsGateFourEvenWithCorrectHash`
- `reSignedBuildWithDifferentCertificateFailsIdentityGate`
- `unsignedOrInvalidPolicyFailsManifestGate`
- `injectedExtraAssetFailsInventoryGate`
- `unexpectedNativeLibraryFailsNativeGate`
- `changedCoreDecoderFailsCoreGate`
- `releaseWithoutPolicyFailsClosed`
- `debugWithoutPolicyReportsBypassAndNeverPass`
- `debugWithPolicyRunsButIsNeverReportedAsVerified`
- `missingProtectedDeclarationFailsResourceGate`
- `goldenCoreVectorsMatchShippedProtocol`
- `goldenCoreVectorsRejectDifferentProtocolConstants`
- `signedEnvelopeVerifiesAndDetectsTampering`
- `hmAcOrUnknownSignatureAlgorithmIsRejected`

Release policy shipped inside the APK:

| Field | Value |
|-------|-------|
| Format / algorithm | 1 / `SHA256withRSA` |
| Signature | 256 bytes, verified with the APK signer certificate |
| Signature result | valid |
| `signerCertSha256` | `c649cb4f…18d098` (matches APK certificate) |
| `expectedSignerSha256` in payload | `c649cb4f…18d098` |
| `debug` | `false` |
| Assets signed | 165 |
| Protected assets | 3 (`ecm_database.json`, `flashpro_profiles.json`, `flashpro_maps/manifest.tsv`) |
| Package / version | `com.revcore.android` / 1.0 |
| Native allow-list | `libandroidx.graphics.path.so` |

The policy JSON inside the APK is byte-identical to the generated release
policy (`SHA-256 fb8c93cf…df`). The detached signature was verified against the
public key of the APK signing certificate by an `apksig`-based Java verifier,
not only by the app at runtime.

Gate names are present in the compiled DEX: APK Identity Guard, UI Encryption
Guard, Resource Protection Guard, ECU Database Guard, Integrity Manifest Guard,
Native Code Guard, Runtime Protection Guard, Final Trust Gate, Core Protection
Guard, Asset Inventory Guard.

Label: source-code verified, automated test verified, APK-package verified.
On-device runtime tamper execution (installing a modified APK and observing the
failure screen) was not performed; no device was attached.

## 5. APK package verification

`apksigner verify --verbose --print-certs`:

- `Verifies`
- v1 scheme: `false` (disabled by design)
- v2 scheme: `true`
- v3 scheme: `true`
- v3.1 / v4: `false`
- Signers: 1
- DN: `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta, ST=DKI Jakarta, C=ID`
- Certificate SHA-256: `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`
- Key: RSA 2048

`aapt2 dump badging` and manifest dump:

- package `com.revcore.android`
- versionCode 1, versionName 1.0
- minSdk 26, targetSdk 36
- no `debuggable` attribute

Archive inspection:

- 168 `assets/` entries = 165 signed inventory entries +
  `integrity_policy.json` + `dexopt/baseline.prof` + `dexopt/baseline.profm`.
- Native libraries: only `libandroidx.graphics.path.so`, for
  arm64-v8a, armeabi-v7a, x86, and x86_64.
- No `.p12`, `.jks`, `.keystore`, `[berkas kunci lokal - tidak dipublikasikan]`, `local-signing.properties`,
  `.pem`, or `.key` entry.
- Zero `BEGIN PRIVATE KEY` / `BEGIN RSA PRIVATE KEY` markers in the APK bytes.
- No `CN=Android Debug` certificate in the exported build.

Exported APK matches the Gradle output byte-for-byte
(`cd8ef5cb…177d` for both files).

## 6. JOB 3 — philosophy section

- Source places "Filosofi RevCore" between Project Credits and Catatan inside
  Hamburger → Pengaturan → Info Aplikasi; no new menu entry.
- Compiled DEX contains: `Filosofi RevCore`, `Built to Inspire`,
  `Not to Impersonate`, and `free to inspire`.
- Existing About content, Indonesian/English localization, and navigation were
  not restructured.

Label: source-code verified, APK-package verified. Visual layout on a device:
not yet verified.

## 7. Release signing and update compatibility

- The build was produced by `tools/revcore-official-release-gui.ps1` through
  `tools/build-official-release.ps1`, using the local PKCS12 keystore. Passwords
  were entered in the local dialog and never printed or written.
- The build script verified v2 + v3 signatures, package name, non-debuggable
  status, and the absence of key material before exporting.
- The archived 151126 builds
  (`dist/archive/RevCore build 151126_121526.apk` and `_122627.apk`) carry the
  same official certificate, so they are update-compatible with this build.
- `dist/RevCore build 141126.apk` is signed with the Android Debug certificate
  (`b1305d1a…922a`). It is not an official build and cannot be updated in place
  by this APK; installing the official build over it requires uninstalling the
  debug installation first. The file was left untouched, not deleted or renamed.
- Application ID is unchanged (`com.revcore.android`).

## 8. Passed / failed / not verified

Passed:

- 214 unit tests (0 failures, 0 errors, 0 skipped).
- Debug and Release compilation; R8 release build.
- APK v2/v3 signature, official certificate, package identity,
  non-debuggable manifest.
- Signed policy structure and RSA signature, asset counts, protected set.
- APK asset inventory, native allow-list, key-material scan.
- JOB 1 and JOB 3 package content.

Failed: none.

Not yet verified (no device attached):

- Physical USB/FTDI adapter detection and K-Line initialization.
- ECU identification, Live Data, DTC read/clear against a real ECU.
- Honda Beat K1A / Beat Deluxe 2020 AFR behaviour on hardware.
- Installing the APK and observing the integrity failure screen after a
  deliberate re-sign or asset modification.
- Visual review of the About section and Live Monitor notice on a phone.

## 9. Known limitations and tooling notes

- Local anti-tamper is a deterrent, not an impossibility. A patched, re-signed
  APK can remove checks; the certificate pin makes it distinguishable.
- Root, KernelSU, custom ROMs, USB debugging, and debuggers are deliberately
  not blocked.
- The GUI launcher records only `SUCCESS` in
  `dist/.official-release-status.txt`; it does not capture the build script's
  `Write-Host` summary. The APK path, hashes, and certificate in this report
  were verified independently with `apksigner`, `aapt2`, and archive tooling.
- No key value, password, or key-pack content appears in this report or in the
  build artifacts.

## 10. Post-release regression fix — Gate 10

A MIUI/HyperOS device reported ten framework/theme assets as “additional” in
Gate 10. The exported APK contained none of them; the build-time policy (165)
and APK archive (165) matched exactly. The cause was the runtime enumeration
domain, not packaging.

Fix:

- Gate 10 now reads `assets/` file entries from the installed APK archive
  (`sourceDir` plus split APKs), excluding `dexopt/` and `integrity_policy.json`.
- Framework assets merged into `AssetManager` by the OEM are no longer compared.
- Added, removed, and content-modified APK assets remain detectable; no
  filename allow-list was added.

Tests added:

- `removedAssetFailsInventoryGate`
- `apkInventoryComesFromArchiveEntriesAndExcludesGeneratedFiles`
- `apkInventoryStillDetectsAnAssetInjectedIntoTheArchive`
- `platformMergedFrameworkAssetsAreNotInTheApkInventory`

Totals at that stage: **218 tests, 0 failures, 0 errors, 0 skipped; 20 integrity
tests.** The `dist/` APK had not yet been rebuilt. The hardened rebuild in §11
supersedes those package-level results; one MIUI/HyperOS runtime check remains
outstanding.

## 11. Hardened rebuild (final)

This section supersedes the package-level results in sections 1–10.

| Check | Result |
|---|---|
| APK | `dist/RevCore build 151126.apk`, 39,623,151 bytes |
| SHA-256 | `0072f4cbf8907ceb0c48f8b98ba3cb249cc570808c8beeee05922455e53c89aa` |
| Signature | v2 true, v3 true, one signer, official certificate `c649cb4f…18d098`, RSA 2048 |
| Debuggable | No |
| Key material in APK | None found |
| Policy | schema 1, `com.revcore.android`, `debug=false`, `SHA256withRSA`, signer = APK certificate |
| Asset inventory | 165 policy / 165 APK, 0 extra, 0 missing |
| Protected assets | 3/3 size and SHA-256 match |
| Native libraries | `libandroidx.graphics.path.so` only, matches policy |
| Unit tests | 232 passed, 0 failed, 0 errors, 0 skipped |
| Emulator | Official Release installed on AOSP API 37; dashboard reached; 489 ECM models loaded from verified APK bytes; no integrity failure screen |

The runtime pass is meaningful because a release build exposes verified asset
bytes only when all ten gates PASS. Physical MIUI/HyperOS and root/custom-ROM
testing remain outstanding; no such result is claimed here.

The earlier artifact (`cd8ef5cb…09177d`) is preserved at
`dist/archive/RevCore build 151126_134108.apk`; it must not be distributed as the
hardened build.
