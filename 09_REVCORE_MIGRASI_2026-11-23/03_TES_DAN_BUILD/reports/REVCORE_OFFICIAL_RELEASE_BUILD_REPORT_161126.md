# REVCORE — OFFICIAL RELEASE BUILD REPORT

Artifact: `RevCore build 161126.apk`
Build date: 2026-11-16 (local time, UTC+07:00)
Build window: 06:31:32 → 06:34:40 (+07:00)
Report ID: `REVCORE-OFFICIAL-BUILD-161126`
Authorization: owner authorization "REVCORE — AUTHORIZATION FOR OFFICIAL SENTINEL RELEASE BUILD".

---

## 1. PRE-BUILD CHECKS

| # | Check | Result |
|---|---|---|
| 1 | Latest Sentinel implementation and compliance reports reviewed | Done: `REVCORE_FINAL_INTEGRITY_COMPLIANCE_REPORT.md`, traceability revision 4, test/lifecycle reports |
| 2 | All 308 JVM tests pass | **PASS** — 308 tests, 0 failures, 0 errors, 0 skipped (35 classes), rerun immediately before build |
| 3 | Debug and Release Kotlin compilation | **PASS** — `:app:compileDebugKotlin`, `:app:compileReleaseKotlin` BUILD SUCCESSFUL |
| 4 | Unresolved compilation errors / release blockers | **None found** |
| 5 | `applicationId` unchanged | **PASS** — `com.revcore.android` (`app/build.gradle.kts:66`) |
| 6 | ECU/FTDI/K-Line/DTC/Live Data/MAP/Tuning untouched | **PASS** — only eight integrity/startup/UI files changed after 05:30; protocol, comm, database, tuning, live-data sources keep 2026-11-15 timestamps |
| 7 | No check silently skipped | **PASS** — every check above has a recorded result |

The release-blocking decision point was not reached; no issue required stopping before signing.

---

## 2. BUILD AND SIGNING PIPELINE

Executed through the existing secure local flow `tools/revcore-official-release-gui.ps1` →
`tools/build-official-release.ps1`:

1. Owner entered the PKCS12 passwords in the local masked WinForms dialog. Values stayed in the
   signing process only; nothing was printed, written, or sent to chat.
2. `:app:assembleRelease` with R8 and resource shrinking (`-Prevcore.publishDist=false`).
3. R8 mapping guard: the eight security Gson models are kept by name (PASS).
4. `apksigner verify` before injection: v2 + v3 true, official certificate.
5. Signed policy verification: envelope `SHA256withRSA`, 165 assets (3 protected), native library
   inventory matches.
6. UI/DEX manifest generated from the **final packaged bytes** (19 UI entries, 1 DEX entry,
   6 compiled XML structures), signed with the official key, injected into the APK.
7. APK re-aligned (zipalign 4-byte) and re-signed with the official key; v1 disabled by design,
   v2 + v3 enabled.
8. Final verification after re-signing: signature, certificate, `debuggable=false`, package,
   policy bytes unchanged, manifest signature and every entry hash verified.
9. Export to `dist/` with a same-day archive guard. No 161126 artifact existed, so nothing was
   archived and nothing was overwritten.

---

## 3. ARTIFACT FACTS

| Property | Value |
|---|---|
| Path | `dist/RevCore build 161126.apk` |
| Size | 39,689,642 bytes |
| SHA-256 | `b2b6c5bd68cc9c727c2babe0c510b928a6135fa7ef5e9d8847275887c8647432` |
| Checksum file | `dist/RevCore build 161126.sha256.txt` (matches) |
| Exported hash equals built APK hash | Yes (`app/build/outputs/apk/release/app-release.apk`) |
| Package | `com.revcore.android` |
| versionCode / versionName | `1` / `1.0` (unchanged; no version bump requested) |
| `debuggable` | `false` (attribute absent) |
| minSdk / targetSdk / compileSdk | 26 / 36 / 36 |
| Signing schemes | v2 true, v3 true, v1 false (by design for minSdk 26) |
| Signer | 1 signer, RSA 2048 |
| Certificate DN | `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta, ST=DKI Jakarta, C=ID` |
| Certificate SHA-256 | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| Public key SHA-256 | `debd3e8ed682223b4f1cd169cbf78a1e2bc07d8834ed01508a38e7c4f4f44b29` |
| Policy signer pin | `c649cb4f…18d098` (`SHA256withRSA`) |
| UI manifest signer | `c649cb4f…18d098` (`SHA256withRSA`) |
| Key material inside APK | none (`.p12`, `.jks`, `.keystore`, key pack, local signing properties) |
| Archive parse | `unzip -t` clean; `aapt2 dump` succeeds |

Previous artifacts preserved: `RevCore build 141126.apk` and `RevCore build 151126.apk`
(SHA-256 `1128af1f…fa86aa`) remain untouched in `dist/`.

---

## 4. SENTINEL INCLUSION EVIDENCE

The exported DEX contains the owner-mandated UI strings (independent grep of `classes.dex`):

`PEMERIKSAAN INTEGRITAS REVCORE`, `PEMERIKSAAN INTEGRITAS GAGAL`,
`Pemeriksaan Integritas Berhasil`, `GERBANG `, `LANJUTKAN`, `KELUAR`, and all seven headings:
`ANOMALI IDENTITAS UI TERDETEKSI`, `ANOMALI SIGNING APK TERDETEKSI`,
`ANOMALI STRUKTUR FILE TERDETEKSI`, `ANOMALI DATABASE TERDETEKSI`,
`ANOMALI MANIFEST APK TERDETEKSI`, `ANOMALI NATIVE CODE TERDETEKSI`,
`ANOMALI KOMPONEN INTI TERDETEKSI`.

The previous artifact `RevCore build 151126.apk` contains none of these strings, so the new APK
demonstrably includes this task's code rather than reusing the older artifact.

---

## 5. FINAL TEST SUMMARY

| Suite | Result |
|---|---|
| JVM unit tests | **308 / 308 pass**, 0 failed, 0 errors, 0 skipped |
| Debug Kotlin compilation | PASS |
| Release Kotlin compilation | PASS |
| Release assembly + R8 | PASS (2 m 56 s) |
| Signature verification | PASS (v2 + v3) |
| Instrumented / emulator / device | **0 executed** |
| Physical ECU / adapter | **0 executed** |

Command used for tests:

```
gradle.bat :app:testDebugUnitTest :app:compileDebugKotlin :app:compileReleaseKotlin --offline --console=plain
```

---

## 6. KNOWN LIMITATIONS AND UNRESOLVED RISKS

1. **No device evidence.** The APK has not been installed or launched on a phone in this task.
2. **No ECU evidence.** FTDI, K-Line, identification (incl. K1AA-N02), Live Data, DTC, MAP, and
   Tuning are preserved in source but not regression-tested on hardware.
3. **Compose UI unproven at runtime.** Ten rows, success window, failure window, and `KELUAR`
   teardown are source-verified and JVM-model-tested only.
4. **Sanitizer is pattern-based.** A future evaluator phrase not in the replacement list could
   appear in the failure window until the list is extended.
5. **Keystore behavior on the device** (marker HMAC alias, record GCM) is not exercised by this
   build; it is exercised at first launch.
6. **Tamper resistance is not proven** against a rooted or repackaged environment; the design is
   root/ROM-agnostic, not a hardware root of trust.
7. **Same versionCode (1) as the previous build.** Installing over 151126 requires `adb install -r`;
   Android may refuse a downgrade only if the installed build has a higher versionCode, which it
   does not.
8. **Update transition:** the previous install has a trust pair bound to the old build identity.
   Expect a FULL examination on first launch after the update; that is correct behavior.

A successful signed build is **not** proof that Sentinel is fully verified. It proves compilation,
packaging, signing, and static artifact consistency only.

---

## 7. POCO X3 NFC — INSTALLATION AND TESTING CHECKLIST

Device: Poco X3 NFC (MIUI, Android 12/13). Do not uninstall the current RevCore automatically.

Preparation

1. Enable Developer options and USB debugging.
2. Connect USB, authorize the computer (`adb devices` shows `device`).
3. Record the installed version: `adb shell dumpsys package com.revcore.android | findstr versionCode`.

Install (update in place)

4. `adb install -r "dist\RevCore build 161126.apk"`.
5. If installation fails with a signature mismatch, **stop and report**; do not uninstall.
6. Launch RevCore.

Examination

7. Expect `PEMERIKSAAN INTEGRITAS REVCORE` with exactly ten rows `GERBANG 1…10 - MENUNGGU`,
   changing to `MEMERIKSA` and then `TERVERIFIKASI` in order 1–7, 9, 10, 8.
8. No gate names, hashes, or technical detail may appear in those rows.
9. On success: `Pemeriksaan Integritas Berhasil`, one `LANJUTKAN`. Confirm the app does not
   continue by itself before the press.
10. Press `LANJUTKAN`; confirm the existing loading screen and app start.
11. Close and reopen (process alive) — no repeated examination. Force stop and reopen — lightweight
    validation, no examination.
12. Clear App Data, launch again — full visible examination, then fresh success.

Failure-path spot check (only on a disposable test install)

13. Modify a protected asset or re-sign a copy and install it on a spare device/profile; expect
    `PEMERIKSAAN INTEGRITAS GAGAL`, only the affected gates listed, matching anomaly headings,
    and a single `KELUAR` that closes the app. Never test this on the daily-use install.

ECU smoke test (adapter connected, after trust)

14. Adapter detection, K-Line init, identification (including K1AA-N02 if the bike matches),
    Live Data, DTC read/clear, MAP view, Tuning screen. Record results; do not assume pass.
15. Rotate the screen during the examination and on the dashboard; no duplicate examination and no
    dashboard before the trust decision.

Report back: screenshots of the examination, success window, version screen, and any failure text,
plus `adb logcat -d -s RevCoreIntegrity MainViewModel` output.

---

## 8. AUTHORIZATION CONFIRMATION

- Official Release APK built: **yes**, under this authorization.
- Signed with the existing local official key: **yes**.
- Exported to `dist/`: **yes**.
- Published or uploaded anywhere: **no**.
- Keys, keystore, or passwords printed, stored in source, or transmitted: **no**.
- Physical-device, ECU, or Sentinel end-to-end verification claimed: **no**.
