# REVCORE — AI MIGRATION START HERE

**Backup date:** 2026-11-23 (Asia/Jakarta)
**Owner:** Otoy Sullivan (credit: `otoy`)
**Prepared by:** Kodir / Qoder, acting as executor only
**Status:** backup prepared and hash-verified; **owner acceptance of the migration is still pending**

Read this file before touching any RevCore source. It is the entry point for a new AI
or developer. It explains what RevCore is, why Build 161126 is the rebuild baseline,
what lives in this backup, what Gate 10 and the signing identity do, what is proven and
what is not, and which actions are forbidden without Otoy's explicit approval.

---

## 1. What RevCore is

RevCore is an Android application for Honda motorcycle PGM-FI diagnostics:

- Package: `com.revcore.android`
- Version: `1.0` (`versionCode 1`) — unchanged since the first builds
- Language: Kotlin + Jetpack Compose
- SDK: minSdk 26, targetSdk 36, compileSdk 36
- Transport: USB-OTG serial, FTDI FT232R / K-Line at 10400 8N1
- Purpose: read ECU identity, Live Data and DTC; inspect MAP data; reconstruct and
  (only with explicit confirmation) exercise Keyless and remap procedures
- Philosophy (owner-authored): **“RevCore — Jembatan untuk belajar, bukan pagar untuk
  membatasi pengetahuan.”**

RevCore is not a clone or rebrand of another product. External technical identifiers
that must remain for protocol accuracy (`FTDI`, `K-Line`, `Keihin`, `Shindengen`,
`Honda PGM-FI`, asset file names such as `revcore_profiles.json`) are preserved
deliberately. Internal names have been migrated to RevCore. See
`REVCORE_PROJECT_IDENTITY.md`.

---

## 2. Why Build 161126 is the baseline

Build `161126` (16 November 2026) was the last official, signed release produced before
the later rework. It is the rollback baseline for the 22 November rebuild because:

1. It was a verified official APK with the official certificate, signed integrity
   policy, UI/DEX manifest, and ten-gate runtime.
2. Later source work (Build 221126 and associated experiments) had diverged; the owner
   ordered a return to the 161126 application source rather than continuing from 221126.
3. The rebuild had to add ten requested works without weakening Gate 10, signing, or
   identity.

The baseline APK is:

| Item | Value |
|---|---|
| File | `work/revcore/dist/RevCore build 161126.apk` |
| Size | 39,689,642 bytes |
| SHA-256 | `b2b6c5bd68cc9c727c2babe0c510b928a6135fa7ef5e9d8847275887c8647432` |
| Certificate SHA-256 | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| Build status | `SUCCESS 2026-11-16T06:34:40` |

**Do not overwrite this baseline with newer source.** The baseline source tree and the
current working tree are stored separately in `01_BASELINE_161126/` and
`02_CURRENT_WORKING_STATE/`.

---

## 3. What changed after 161126

The current working tree is branch `rebuild-161126` at commit `1cc9120` plus uncommitted
work. The rebuild implemented ten ordered works and four final UI corrections. Summary:

| # | Work | State |
|---|---|---|
| 1 | Unified Live Data + actuator table, no Role Table | Implemented, unit-tested |
| 2 | “Prakiraan AFR” from researched O2 formula | Implemented, unit-tested, not a measured AFR |
| 3 | Real `.txt` log share, honest “Buka Lokasi”, structured events | Implemented |
| 4 | Full Keyless ID, Renew/Reset gated by a verified read | Implemented; hardware pending |
| 5 | Tuning/Remap overhaul with FTDI; typed confirmations | Implemented; write hardware not validated |
| 6 | Protocol Manager + USB stability | Implemented, unit-tested |
| 7 | Keep screen on in all menus | Implemented |
| 8 | Gate 10/trust → 2 s hold → loading → menu on every path | Implemented, unit-tested |
| 9 | DTC layout: Scan top / list middle / Hapus bottom | Implemented |
| 10 | Internal identity audit to RevCore | Implemented; external identifiers preserved |

Final corrections on 22 November:

- startup hold applies to **all** paths, including failure and trust-record reuse;
- Live Data stays dynamic, retains the last reading with an explicit state, and keeps
  the AFR estimate visible with its caveat;
- Keyless order is 1–5 with copy popup; protocol follows the ECM family; an ACK is not
  proof; success requires read-back;
- Tuning layout references FlashPro but carries RevCore identity; MAP selection/export
  remain; no false “Write BIN” claim while write hardware is unproven.

Detailed evidence is in `07_PROJECT_HISTORY/REBUILD_WORKS_1_10.md` and
`07_PROJECT_HISTORY/UI_CORRECTIONS_20261122.md`.

Current official Release artifact:

| Item | Value |
|---|---|
| File | `work/REVCORE_KEYLESS_20261120/RevCore_work_live_keyless_20261120/dist/RevCore build 221126.apk` |
| Size | 39,841,086 bytes |
| SHA-256 | `cb5892a3b956c566e2f9cb3f0ae77f84488f1cd4902803b0ca756298720ba556` |
| Certificate SHA-256 | `c649cb4f…18d098` (same official identity) |
| Build status | `SUCCESS 2026-11-22T22:09:32` |

The APK is a **Release** build, not a debug substitute. It has not been validated on a
physical phone or ECU in the recorded evidence.

---

## 4. Where the source actually is

| Purpose | Original location | Backup copy |
|---|---|---|
| Baseline project, pre-rebuild | `C:\Users\Lord Otoy\Documents\Qoder\work\revcore` | `01_BASELINE_161126/project` |
| Baseline APK and reports | same, `dist/` | `01_BASELINE_161126/dist` |
| Current working repository | `C:\Users\Lord Otoy\Documents\Qoder\work\REVCORE_KEYLESS_20261120\RevCore_work_live_keyless_20261120` | `02_CURRENT_WORKING_STATE/project` |
| Current APKs and release status | same, `dist/` | `02_CURRENT_WORKING_STATE/dist` |
| Git history | current repository `.git` | `03_GIT_HISTORY/revcore_repository.bundle` |
| Gate 10 source, policies, tests | both trees | `04_GATE10_SECURITY` |
| Private signing identity | folder kunci lokal pemilik | **di luar paket ini** (lokasi tidak dipublikasikan) |
| Owner research archive | `C:\Users\Lord Otoy\Desktop\Bahan Otopsi.zip` | `06_RESEARCH_ARCHIVE/archive/Bahan Otopsi.zip` |

The project has **no Gradle wrapper**. Use the installed Gradle 9.6.0 distribution:

```bash
JAVA_HOME="C:/Program Files/Android/Android Studio/jbr" \
  ~/.gradle/wrapper/dists/gradle-9.6.0-bin/42k10rwplmzkhuboz9kdazi7s/gradle-9.6.0/bin/gradle \
  :app:testDebugUnitTest --offline
```

Other verified tasks: `:app:compileDebugKotlin`, `:app:assembleDebug`. Official Release
must go through `tools/revcore-official-release-gui.ps1`; never create a release by
calling Gradle with invented credentials.

---

## 5. Reading order

1. This file.
2. `REVCORE_PROJECT_IDENTITY.md` — name, credit, philosophy, what must not be rebranded.
3. `OWNER_AUTHORITY_AND_RULES.md` — authority, prohibitions, report format, evidence rules.
4. `04_GATE10_SECURITY/GATE10_EXPLAINED.md` — how the ten gates and trust record work.
5. `05_PRIVATE_KEY_INVENTORY/KEY_INVENTORY.md` and `SIGNING_PROCEDURE.md` — signing
   identity without exposing secrets.
6. `07_PROJECT_HISTORY/REVCORE_PROJECT_HISTORY.md` — the full project journey.
7. `08_ARCHITECTURE_AND_FEATURES/` — how each subsystem is built.
8. `10_KNOWN_ISSUES/` — unfinished work, evidence limits, hardware status.
9. `11_MIGRATION_MANIFEST/BACKUP_VERIFICATION_REPORT.md` — what was verified.

Do not start by rewriting code. Start by reading these files and checking the actual
source. The backup contains evidence, not a mandate to “clean up” the project.

---

## 6. Gate 10 in one page

Gate 10 is **not a feature toggle and not an obstacle**. It is the app's trust boundary:

- Build time generates `assets/integrity_policy.json` from the **final packaged assets**,
  signs it with the official release key, and injects a signed
  `assets/ui_integrity_manifest.json` before re-signing the APK.
- Runtime runs nine evidence gates in order `1, 2, 3, 4, 5, 6, 7, 9, 10`, then Gate 8
  (`FINAL_TRUST`) as the aggregate. Every release gate must PASS; `BYPASS` and
  `NOT_CHECKED` never count as pass.
- A passing full exam can commit an installation marker and an algoritma terautentikasi (detail tidak dipublikasikan) trust record
  sealed by Android Keystore. Later launches re-verify the record's bindings; a mismatch
  falls back to a full exam, never to a silent pass.
- The owner-mandated startup order is: exam or trust record → static 2-second hold →
  “Loading RevCore” → main menu. Failure paths also pass the hold and loading screen.

**Never** disable, weaken, bypass, rename a failing gate to PASS, remove a protected
asset from the policy, or regenerate the trust record manually. If a source change is
official, rebuild through the official script so the policy and manifests are
regenerated and signed; if the change is not official, do not ship it. Full details:
`04_GATE10_SECURITY/GATE10_EXPLAINED.md`.

---

## 7. Signing identity and “.key” files in one page

There are **no files with a `.key` extension** in either project, on the Desktop key
folder, or in the owner's Documents tree. The operational signing identity is:

- `[berkas kunci lokal - tidak dipublikasikan]` — PKCS12 keystore, alias `[alias kunci lokal]`
- `[berkas kunci lokal - tidak dipublikasikan]` — legacy/JKS copy of the same identity
- `[berkas kunci lokal - tidak dipublikasikan]` — local key pack
- `[catatan kunci lokal - tidak dipublikasikan]` — local notes

Passwords are **never** in this backup. The AI package contains only names, sizes,
SHA-256 hashes, and usage/procedure notes. The byte-identical private backup is outside
the AI package at:

`[lokasi cadangan kunci privat - tidak dipublikasikan]`

The new AI must use the original files, must not regenerate or rename keys, and must
never ask Otoy to type a password into chat. Otoy enters the password locally in the
existing masked WinForms dialog. See `05_PRIVATE_KEY_INVENTORY/`.

---

## 8. What is proven, and what is not

**Proven by executed JVM tests:** gate decisions and prerequisites, staged order,
trust-record/UI-manifest container logic, protocol parsing, Keyless frame/decoder logic,
remap codecs/checksums/guards, unified Live Data rows, AFR estimator, startup timing,
database/map parsing and identity matching.

Latest recorded run (2026-11-22, before the final APK):

- 55 suites, 457 tests, 0 failures, 0 errors, 0 skipped
- baseline reference: 41 suites, 353 tests, 0 failures

**Not proven:** anything requiring a physical phone, USB adapter, K-Line bus, or ECU.
Gate 10 runtime/Keystore on a real OEM ROM, MIUI/HyperOS USB quirks, Live Data on a
running engine, DTC clear, Keyless Renew/Reset, and remap read/write/verify are still
owner/device validation items. “Build successful” is not “feature successful”.

The tenant may not claim physical ECU validation. See `10_KNOWN_ISSUES/`.

---

## 9. Repository state you are inheriting

- Branch: `rebuild-161126`
- HEAD: `1cc9120` — “docs(rebuild): record the verified Build 161126 baseline and research archive”
- Rollback commit: `656bed2` — “rollback: restore app source to Build 161126”
- Baseline documentation commit: `1cc9120`
- Other branch: `master` at `e761002`
- No remotes, no tags, no stash in the working repository.
- Working tree: 233 `git status --short` entries (directories collapsed), comprising
  185 staged rename/bookkeeping entries, 46 unstaged tracked files, and 59 untracked
  files. The exact patches and file hashes are in `03_GIT_HISTORY/`.

Nothing in this backup was committed, pushed, re-versioned, re-signed, or published.
The backup process did not alter source or repository history.

---

## 10. Immediate rules for the successor AI

1. Otoy is the highest authority. The AI is an analysis and implementation tool, not the
   owner.
2. Do not change the credit `otoy`, package name, version, theme, feature set, signing
   identity, or Gate 10 without an explicit owner order.
3. Do not commit, push, change branch, delete history, change remotes, publish, or create
   a release without explicit approval. This backup itself is not approval.
4. Do not invent protocols, checksums, seed/key algorithms, EEPROM addresses, or test
   results. If evidence is missing, say “unknown” and separate experiments honestly.
5. Read the source before editing. Report actual old code and actual replacement code,
   not illustrative snippets.
6. Do not treat this backup as “migration accepted”. The final decision belongs to Otoy.

The condensed report format and evidence labels are in
`00_READ_FIRST/OWNER_AUTHORITY_AND_RULES.md`.

---

## 11. Backup verification status

- Baseline project: 2,622 files, byte-identical to the original (SHA-256 diff = 0).
- Current project + dist: 746 files, byte-identical to the original (SHA-256 diff = 0).
- Research archive: `Bahan Otopsi.zip` byte-identical, SHA-256
  `b9db1554b41952874f2121059724410f4569ce073952c601424b5fc59ffae1b9`.
- Private key backup: four files byte-identical by `cmp`; ACL restricted to the owner,
  SYSTEM, and Administrators.
- Explicit exclusions: regenerable Gradle/build caches, `.git` working copy (replaced by
  a verified bundle), old APK archive folders, and `debug.keystore` copies. These are
  listed with reasons in `11_MIGRATION_MANIFEST/EXCLUSIONS_AND_LIMITS.md`.

The detailed counts and comparison commands are in
`11_MIGRATION_MANIFEST/BACKUP_VERIFICATION_REPORT.md`.
