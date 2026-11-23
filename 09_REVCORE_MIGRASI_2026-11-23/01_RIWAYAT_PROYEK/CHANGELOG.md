# REVCORE — CHANGELOG

**Sifat:** ringkasan kronologis. Sumber utama tetap laporan di
`09_TESTS_AND_BUILD/reports/` dan `02_CURRENT_WORKING_STATE/project/delivery/`.

Legenda status:

- **LAPORAN** — peristiwa tercatat di dokumen, belum tentu diuji ulang.
- **BUILD** — artefak APK dihasilkan.
- **TEST** — unit test dijalankan.
- **PERANGKAT** — hanya boleh dipakai bila Otoy sudah menguji di perangkat.
- **BELUM** — belum ada bukti.

---

## 2026-10-27 — Jejak proyek AI Studio

- `README.md` dan folder `.aistudio` menandai asal proyek Google AI Studio.
- Rencana awal: perbaikan Live Telemetry dan Clear DTC/MIL Honda.
- Status: LAPORAN.

## 2026-11-04 — One-pass Live Telemetry dan DTC

- Laporan `docs/revcore_one_pass_report.md`.
- Fokus: frame K-Line, ACK pendek, nilai semu, dan kode hantu `MIL#193-128`.
- Status: LAPORAN; sebagian diuji software.

## 2026-11-05 — BikeStudio dan Tuning progress

- `BIKESTUDIO_INTEGRATION_REPORT.md`: integrasi suplemen non-destruktif.
- `TUNING_PROGRESS_IMPLEMENTATION_REPORT.md`: Live Tuning Log dan progres nyata.
- Status: LAPORAN.

## 2026-11-08 — NO.1–NO.4 dan MAP orisinal

- `docs/revcore_no1_no4_final_report.md`:
  - NO.1 identitas ECU,
  - NO.2 TPS/KOEO,
  - NO.3 database Indonesia 117 record,
  - NO.4 MAP catalog/decoder/editor dan gerbang flashing.
- `docs/revcore_original_map_integration_report.md`: pustaka MAP orisinal.
- `docs/revcore_tuning_replacement_report.md`: penggantian alur Tuning.
- Status: LAPORAN; test offline.

## 2026-11-11 — Master task terpadu

- `docs/revcore_master_task_final_report.md`.
- Penyatuan Tuning, navigasi, editor, dan build.
- Status: LAPORAN; test offline.

## 2026-11-13 — Redesign UI Remap

- `docs/revcore_remap_ui_redesign_report.md`.
- Perubahan besar pada hierarki Remap dan editor.
- Status: LAPORAN.

## 2026-11-14 — Audit keamanan, signing, rollback selektif, build 141126

- `REVCORE_SECURITY_AUDIT_PHASE_0.md`: menemukan build "release" masih
  debug-signed, `debuggable`, dan memakai `debug.keystore`.
- `REVCORE_LOCAL_SIGNING_SETUP_REPORT.md`: prosedur signing lokal.
- `docs/revcore_selective_rollback_report.md`: redesign Remap di-rollback ke
  snapshot sebelum 2026-11-13 22:29 WIB.
- `dist/RevCore build 141126.apk`: build debug-signed; **bukan** baseline resmi.
- Status: LAPORAN + BUILD.

## 2026-11-15 — Build 151126 dan pengerasan integritas

- `dist/RevCore build 151126.apk`.
- `REVCORE_UI_INTEGRITY_AUDIT.md`, `REVCORE_UI_INTEGRITY_IMPLEMENTATION.md`,
  `REVCORE_UI_INTEGRITY_REPORT.md`.
- `REVCORE_SECURITY_TEST_REPORT.md`, `REVCORE_ROOT_COMPATIBILITY_TEST_REPORT.md`,
  `REVCORE_ROM_COMPATIBILITY_AUDIT.md`.
- Trust record `[berkas trust eksternal - tidak dipublikasikan]` dan installation marker mulai dipakai.
- Status: BUILD + TEST; perangkat belum diklaim.

## 2026-11-16 — Build resmi 161126

- `dist/RevCore build 161126.apk` — 39.689.642 byte; SHA-256
  `b2b6c5bd68cc9c727c2babe0c510b928a6135fa7ef5e9d8847275887c8647432`.
- `REVCORE_OFFICIAL_RELEASE_BUILD_REPORT_161126.md`.
- `REVCORE_SIGNING_VERIFICATION_161126.md`: sertifikat resmi
  `c649cb4f…18d098`.
- `REVCORE_SENTINEL_IMPLEMENTATION_REPORT.md`,
  `REVCORE_SENTINEL_REQUIREMENTS_TRACEABILITY.md`.
- Perbaikan Gate 10 untuk positif palsu inventaris aset.
- Status: BUILD + TEST; baseline rebuild.

## 2026-11-20 — Working copy Live Data + Keyless

- Working copy `work/REVCORE_KEYLESS_20261120/RevCore_work_live_keyless_20261120`.
- Audit H-Diag, FlashPro Keyless, source RevCore, FTDI K-Line; delapan dokumen
  audit.
- Implementasi perbaikan Live Data dan Keyless ID; build Release debug lokal.
- Status: LAPORAN + BUILD lokal; bukan official release.

## 2026-11-22 — Rollback, sepuluh rebuild, empat koreksi, build 221126

### Rollback

- `656bed2` — source aplikasi dikembalikan ke Build 161126.
- `1cc9120` — baseline dan arsip riset didokumentasikan.
- Branch `rebuild-161126`; `master` menyimpan pekerjaan 221126 (`e761002`).

### Sepuluh pekerjaan rebuild

- Unified Live Data, AFR, log/event, Keyless, Tuning/Remap, Protocol Manager,
  keep-screen, startup Gate 10, UI DTC, identitas RevCore.
- Laporan: `delivery/rebuild_161126_20261122/CHANGE_REPORT.md`.
- Status: implementasi software + test; validasi kendaraan BELUM.

### Empat koreksi absolut

- Startup hold 2 detik + loading di semua jalur.
- Live Data hanya parameter valid + retensi + AFR setelah O2 valid.
- Keyless urutan 1–5 dan read-gated.
- Tuning mengikuti alur FlashPro dengan identitas RevCore.
- Laporan: `ABSOLUTE_ORDER_REPORT.md`,
  `CORRECTION_ORDER_REPORT_20261122.md`,
  `UI_CORRECTIONS_20261122.md`.

### Build resmi 221126

- `dist/RevCore build 221126.apk` — 39.841.086 byte; SHA-256
  `cb5892a3b956c566e2f9cb3f0ae77f84488f1cd4902803b0ca756298720ba556`.
- Ditandatangani melalui dialog Owner; diverifikasi v2/v3, cert resmi,
  package/versi, non-debuggable.
- Gate 10: 165 path policy = 165 aset APK; 0 mismatch.
- Status: BUILD + TEST; perangkat BELUM.

### Perubahan status RENEW

- Commit `329405a` mengaktifkan RENEW sebagai zero-write reset.
- `KeylessRenewPlan.RENEW_SUPPORTED = true` di source current.
- Dokumen delivery lama masih `false`; drift dicatat, tidak dihapus.

## 2026-11-23 — Backup migrasi AI

- Paket `REVCORE_AI_MIGRATION_BACKUP/` disusun.
- Baseline 161126, current working state, Git history, Gate 10, private key
  inventory, research archive, history, architecture, tests, known issues, dan
  manifest.
- Status: backup + verifikasi hash; **bukan** persetujuan migrasi.

---

## Catatan versi

- `versionName` tetap `1.0`; `versionCode` tetap `1`; package
  `com.revcore.android`.
- Tidak ada perubahan versi/identitas/signing tanpa perintah Otoy.
- Build resmi yang tercatat: 151126, 161126, 221126. Build 141126 adalah
  debug-signed dan tidak boleh dipakai sebagai dasar migrasi.
- Build debug (misalnya `app-debug.apk`) bukan pengganti APK resmi.

**Keputusan akhir milik Otoy.**
