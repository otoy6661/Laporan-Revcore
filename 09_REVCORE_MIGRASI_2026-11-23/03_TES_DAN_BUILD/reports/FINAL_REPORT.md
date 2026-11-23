# REVCORE REBUILD 161126 — FINAL REPORT (§IX)

Order: `MASTER WORK ORDER — REVCORE REBUILD: ROLLBACK BUILD 161126 + IMPLEMENTASI 10 TUGAS TERPADU`
Owner: Otoy Sullivan (`otoy`) — Executor: Kodir
Date: 2026-11-22
Baseline: Build 161126 (`656bed2`, documented by `1cc9120`); no 221126 source was used as the foundation.
Branch: `rebuild-161126`
Repository: https://github.com/otoy6661/Revcore-Project (working copy: `RevCore_work_live_keyless_20261120`)

## Result

All ten ordered tasks are implemented in the working tree. The project compiles, all unit tests pass, and a debug APK assembles. Nothing was pushed, published, zipped, re-versioned, or signed with a release key.

| # | Task | Status | Primary evidence |
|---|---|---|---|
| 1 | Unified Live Data table | Implemented | `core/comm/UnifiedLiveTable.kt`; `LiveDataScreen` single table; `UnifiedLiveTableTest` |
| 2 | "Prakiraan AFR" from researched formula | Implemented | `core/comm/AfrEstimator.kt`; always-visible card with caveat; `AfrEstimatorTest` |
| 3 | Log fixes (real `.txt`, honest "Buka Lokasi", structured events) | Implemented | `FileProvider` share, guarded folder intents, per-menu `CommEvent` statuses |
| 4 | Full Keyless ID + Renew/Reset gating | Implemented | `core/keyless/*`, `KeylessScreen`, engine read-back verification, stale-read invalidation |
| 5 | Tuning/Remap overhaul | Implemented | `core/remap/*`, `RemapExecutionGuard`, typed confirmations, separate read-back verification, real action wiring |
| 6 | Protocol Manager + USB stability | Implemented | `ProtocolManager`, engine `requireLiveTransport`/`verifyTransportHealth`, `TRANSPORT_DEGRADED`, ECU-reset session invalidation; `ProtocolManagerTest` (7 tests) |
| 7 | Keep screen on for all menus | Implemented | `MainActivity.onResume/onPause` add/clear `FLAG_KEEP_SCREEN_ON` |
| 8 | Startup gate 10 → confirmation → 2 s loading | Implemented | process-scoped `startupState` render; debug builds also show the confirmation window; `STARTUP_LOADING_MIN_MS = 2000` |
| 9 | DTC layout | Implemented | `DiagnosticsScreen`: Scan top, scrollable list, Hapus DTC bottom |
| 10 | Full identity audit | Implemented | `data/flashpro` → `data/maps`, type/UI/label renames; `IDENTITY_AUDIT.md` |

## Test labels (§VI)

- **Implemented** — code exists for every task above and is wired to UI actions.
- **Build successful** — `:app:compileDebugKotlin` exit 0; `:app:assembleDebug` exit 0.
- **Unit tested** — `:app:testDebugUnitTest` exit 0: **51 suites, 435 tests, 0 failures, 0 errors, 0 skipped** (baseline evidence: 41 suites / 353 tests / 0 failures in `baseline/test-results-baseline`).
- **Integration tested** — not performed against a real adapter/ECU in this session (no hardware attached).
- **Simulated / reconstructed** — Keihin/Shindengen read/write frames and Keyless sequences are `RECONSTRUCTED_FROM_BINARY` / `PROTOCOL_ANALYZED`; unit tests exercise them with recorded/synthetic frames, not a vehicle.
- **Hardware not tested** — USB/FTDI, K-Line, Keyless Renew/Reset, and remap write/verify still require a physical Honda ECU.
- **Owner validation pending** — device validation on Poco X3 NFC (surya, Android 12 / MIUI 14) and the real ECU remains with the owner.

## Build evidence

```text
JAVA_HOME=C:/Program Files/Android/Android Studio/jbr
gradle 9.6.0 (offline wrapper dist)

:app:compileDebugKotlin     exit 0
:app:testDebugUnitTest      exit 0  (435 tests / 0 failures)
:app:assembleDebug          exit 0
```

Artifact (debug build for local testing only; **not** an official release and not published):
`app/build/outputs/apk/debug/app-debug.apk`
size 57,965,567 bytes
SHA-256 `5feb79affcaed28177defda5733df11813b9f9a7721d391de5a4432130582ab2`

Freshness note: the hash corresponds to the build after the package rename and Keyless gating edits. It will change again with any later source change.

## Constraints honoured

- No development continued from Build 221126; no 221126 source was used as a foundation.
- No force push, no Git history deletion, no baseline substitution.
- `bahan otopsi.zip` and `docs/` research assets untouched.
- No proprietary source or third-party asset was copied beyond the owner-provided research/MAP material already in the repository.
- No "Role Table" layer was introduced; one unified table is used.
- Estimates are labelled "Prakiraan AFR" / "Estimated AFR", never measured AFR.
- No success is recorded before a verified outcome (Keyless reset, DTC clear, remap write ACK vs. read-back, identification).
- No decorative/fake Tuning buttons: every action calls a real ViewModel/engine operation.
- Device liveness is checked before transactions; a stale connection object is not treated as an active device.
- No APK publication, no ZIP, no version change, no signing-password request/read/store, no new signing identity.

## Git state (transparent)

- Working tree contains the rebuild changes; **nothing is committed** in this session.
- Unstaged changes: 40 files, +3,898 / −355 lines versus the 161126 baseline.
- Staged rename bookkeeping: 21 files (`git mv` for the `data/flashpro` → `data/maps` package files).
- New untracked source groups: `core/keyless`, `core/remap`, `core/comm/UnifiedLiveTable.kt`, `core/comm/AfrEstimator.kt`, `core/protocol/ProtocolManager.kt`, `data/maps/RemapExecutionGuard.kt`, `ui/keyless`, `ui/RemapOperationState.kt`, `ui/screens/KeylessScreen.kt`, `res/xml/file_paths.xml`, and their tests.
- Reports: `delivery/rebuild_161126_20261122/` (this folder, untracked).
- Awaiting owner acknowledgement before any commit, per the standing commit-after-ack rule.

## Remaining limitations / honest gaps

1. **No physical ECU validation.** Keyless Renew/Reset writes zeros to firmware-identified areas and verifies by read-back, but no vehicle test has confirmed the effect. Remap write/verify is `RECONSTRUCTED_FROM_BINARY` and explicitly not hardware-proven.
2. **Protocol ambiguity.** Where the decompiled firmware did not define a value, the app reports `Unavailable`/`Invalid` rather than inventing data. This can look like "missing" values on a real bike until owner reports arrive.
3. **FTDI/USB behaviour under MIUI.** Permission prompt, detach/attach, and reconnect backoff are implemented and unit-tested at the policy level, but MIUI USB quirks need the owner's device.
4. **UI layout changes (Tasks 7–9)** are compile-verified only; visual confirmation on the Poco X3 is pending.
5. **Tuning log export/share** uses `FileProvider` cache files; sharing behavior depends on the receiving app chooser.
6. **Trust-record reuse path.** When a valid trust record is reused, RevCore runs initialization and the 2 s loading but does not repeat the full 10-gate window, per the existing security policy (the record already represents a prior confirmed FULL exam). If the owner wants the gate window on those launches too, that is a policy decision to confirm.

## Next action requested from owner

Install the debug build on the test device, exercise: startup gate, Info Kendaraan identify, Live Data table + Prakiraan AFR, DTC scan/clear, Keyless read → (only then) Renew/Reset, and Tuning read/verify with the FTDI adapter. Report actual results so the hardware-validation labels can be updated. No commit/push/publication will be made without owner instruction.
