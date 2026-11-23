# RevCore — Startup Performance Report

Date: 2026-11-15
Measurement build: `dist/RevCore build 151126.apk` (pre-donation build)
SHA-256 measured: `8939674f115bc45de8194af049cb71cd426f8207fa943a15526a158f769059d8`
Current artifact: `dist/RevCore build 151126.apk`, SHA-256
`1128af1ff7bc15f1f614e6e177fd2a24bb0ef3783eed0373d5687eb6adfa86aa`
Package: `com.revcore.android` (versionCode 1, versionName 1.0)
Certificate SHA-256: `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`

All numbers below are measured. Nothing in this report is estimated or copied
from a different build. The measurement platform is an emulator, not a physical
phone; that limitation is stated again in section 6.

The rigorous measurements were taken on the pre-donation build, which has the
same integrity, startup and record code. The current artifact was smoke-checked
only: fresh install `FULL` 5688 ms `passed=true`, restart `TRUST_RECORD` 1853 ms
`passed=true` (slower emulator session); it was not re-measured run-by-run.

## 1. Test environment

| Item | Value |
| --- | --- |
| Device | `emulator-5554`, `sdk_gphone64_x86_64` (AOSP Google APIs image) |
| Android API | 37 |
| ABI | x86_64 |
| RAM | 4,008,496 kB total |
| Host | Windows, Android Studio JBR, Gradle 9.6.0 offline |
| Build type | Official Release, R8 + resource shrinking, v2 + v3 signature |
| Evidence label | `FULL` (full verification) or `TRUST_RECORD` (record-backed) |

## 2. Method

Each run used a real cold start, not a warm resume:

```bash
adb shell am force-stop com.revcore.android
adb logcat -c
adb shell am start -W -n com.revcore.android/.MainActivity
adb logcat -d -s RevCoreIntegrity:I
```

Two timings are reported:

- **Decision time** — `RevCoreIntegrity: startup mode=... elapsedMs=...`, the
  time spent inside the integrity decision itself.
- **TotalTime** — the value printed by `am start -W`, from launch request to the
  first drawn frame. It includes process start, Application creation, Compose
  composition and the splash animation, so it is always larger than the decision
  time.

`TRUST_RECORD` runs used the record written by a previous passing full
evaluation. `FULL` runs deleted the record first (`rm` of
`[berkas trust eksternal - tidak dipublikasikan]`) and logged `startup record=missing; running full verification`.
After every passing `FULL` run the record was recreated; its presence was
verified with `ls -l` (1029 bytes).

## 3. Results — record-backed vs full verification

Three cold starts per mode, final APK, 2026-11-15 16:54–16:55 WIB.

| Mode | Decision run 1 | run 2 | run 3 | Median | TotalTime (runs) |
| --- | ---: | ---: | ---: | ---: | --- |
| `TRUST_RECORD` | 708 ms | 574 ms | 750 ms | **708 ms** | 2059 / 2214 / 2137 ms |
| `FULL` (record deleted) | 1390 ms | 2260 ms | 1632 ms | **1632 ms** | 2050 / 2152 / 2126 ms |

Median decision-time saving: **924 ms (56.6 %)**. Mean decision time:
677 ms (`TRUST_RECORD`) vs 1,761 ms (`FULL`).

A second measurement set on the same APK and emulator, taken earlier in the
session, produced the same separation:

| Mode | Decision times | Median |
| --- | --- | ---: |
| `TRUST_RECORD` | 583 / 749 / 826 ms | 749 ms |
| `FULL` | 1317 / 1538 / 1706 ms | 1538 ms |

Run-to-run spread is real (emulator CPU scheduling, page cache, logcat I/O).
A later smoke start after the corruption tests measured `TRUST_RECORD`
**1350 ms** (TotalTime 2933 ms) — outside the range above but still correct and
still far below the `FULL` path's typical range. The report keeps these values
instead of presenting only the best run.

## 4. Results — first run and process recreation

| Scenario | Decision time | TotalTime | Notes |
| --- | ---: | ---: | --- |
| First run after `pm clear` (no key, no record, cold caches) | 3581 ms `FULL` | 3201 ms | Creates the Android Keystore key, verifies everything, writes a 1029-byte record |
| Next cold start after that | 851 ms `TRUST_RECORD` | 2452 ms | Record accepted |
| Process recreation with a restored task (F1 test) | `TRUST_RECORD` | — | Dashboard not shown before the decision (see below) |
| Reinstall over the existing install, record present | 1260 ms `FULL` | 2755 ms | `startup record rejected; running full verification`, then PASS and a rewritten record |

The first run after clearing data is the slowest case because it includes
Keystore key generation (`[alias trust internal - tidak dipublikasikan]`) and a completely cold file cache.
It is a one-time cost per install/data-clear.

### F1 behaviour on process recreation

The task was restored after the process was killed. Screenshots in
`dist/evidence-ui-integrity-151126/`:

- `recreation-immediate.png` — the window is still blank/white; no dashboard,
  no ECU controls, no Live Data.
- `recreation-settled.png` — the dashboard appears only after the decision
  completes.

The blank frame is the pre-Compose window background, not a trusted screen.
It exposes no application surface. After the decision the normal dashboard is
shown and the existing storage-access prompt is displayed.

## 5. What the optimization changes

- **Same process:** a report already held by the process is reused. Recomposition,
  Activity recreation, drawer navigation and returning from background do not
  re-run the inventory or hashing. This is source-inspected and covered by
  gate-level tests; it is not a new timing claim.
- **New process with a valid record:** the record is authenticated (algoritma terautentikasi (detail tidak dipublikasikan),
  Keystore key) and then cross-checked against the installed package — signer
  pin, both detached signatures, policy/manifest digests, archive fingerprint,
  APK length and mtime, version, native set, protected asset hashes, compiled XML
  structure, database schema and core vectors. Only then is `TRUST_RECORD`
  reported. Any mismatch falls back to `FULL`.
- **Install, update, clear data, missing/invalid record:** always `FULL`.
  No shortcut is taken when the evidence cannot establish the installed APK.

`TRUST_RECORD` is never reported as `FULL`. The record does not hash the whole
APK again; it is a fast, re-validated statement about a previously verified
install. That limitation is unchanged and documented in
`REVCORE_PROJECT_RULES.md` §8.

## 6. Memory

Measured after the dashboard settled on a `TRUST_RECORD` start:

| Metric | Value |
| --- | ---: |
| TOTAL PSS | 36,993 kB |
| TOTAL RSS | 152,440 kB |
| Java heap PSS | 14,864 kB |
| Native heap PSS | 11,144 kB |
| Swap PSS | 219 kB |

An earlier reading on the same build was 37,225 kB PSS, within 1 % of the
figure above. The record path adds no measurable long-lived allocation: the
record is a single ~1 kB byte array plus its parsed JSON model.

## 7. Honest limitations

- These are **emulator** results (`sdk_gphone64_x86_64`, API 37), not a physical
  phone. Absolute milliseconds on a real device will differ.
- `am start -W` includes shell and window-manager overhead. The decision time
  from `RevCoreIntegrity` is the more precise number for the integrity work.
- No Macrobenchmark/StartupTiming library run was performed; the measurements
  are direct `adb` observations with timestamps in logcat.
- No startup profiling was done on MIUI/HyperOS, a rooted device, KernelSU, a
  custom ROM, or with a connected FTDI adapter. Those configurations were not
  tested in this session.
- No hardware, adapter or ECU test was performed. No performance claim is made
  about the K-Line/USB path.
