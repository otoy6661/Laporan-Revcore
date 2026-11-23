# REVCORE — SEJARAH PERJALANAN PROYEK

**Pemilik dan pencipta:** Otoy Sullivan (`otoy`)
**Baseline rebuild:** RevCore Build 161126
**Disusun:** 2026-11-23
**Sifat dokumen:** kronologi berbasis dokumen dan source yang tersimpan di paket migrasi ini.

Dokumen ini tidak menggantikan laporan teknis asli. Setiap bagian menunjuk
dokumen bukti yang dapat dibaca langsung di paket backup. Jika suatu peristiwa
hanya diketahui dari laporan, maka peristiwa itu disebut sebagai laporan, bukan
sebagai hasil pengujian ulang.

Build 161126 adalah **baseline rebuild**, bukan satu-satunya versi yang pernah
ada. Versi 221126 dan pekerjaan 20–22 November tetap dicatat di sini dan tidak
dihapus dari sejarah.

---

## Ringkasan kronologi

| Tanggal | Peristiwa | Bukti utama |
|---|---|---|
| 2026-10-27 | Jejak proyek AI Studio: `README.md` dan folder `.aistudio` | `01_BASELINE_161126/project/README.md`, `.aistudio/` |
| 2026-11-04 | Laporan satu-pass: perbaikan Live Telemetry dan Clear DTC/MIL | `docs/revcore_one_pass_report.md` |
| 2026-11-05 | Integrasi BikeStudio dan laporan Tuning progress | `BIKESTUDIO_INTEGRATION_REPORT.md`, `TUNING_PROGRESS_IMPLEMENTATION_REPORT.md` |
| 2026-11-08 | Implementasi NO.1–NO.4 dan integrasi MAP orisinal | `docs/revcore_no1_no4_final_report.md`, `docs/revcore_original_map_integration_report.md`, `docs/revcore_tuning_replacement_report.md` |
| 2026-11-11 | Laporan master task terpadu | `docs/revcore_master_task_final_report.md` |
| 2026-11-13 | Redesign UI Remap | `docs/revcore_remap_ui_redesign_report.md` |
| 2026-11-14 | Audit keamanan Phase 0, setup signing lokal, rollback selektif redesign, build 141126 | `REVCORE_SECURITY_AUDIT_PHASE_0.md`, `REVCORE_LOCAL_SIGNING_SETUP_REPORT.md`, `docs/revcore_selective_rollback_report.md` |
| 2026-11-15 | Build 151126 dan pengerasan Gate 10 / UI integrity | `dist/RevCore build 151126.apk`, `REVCORE_UI_INTEGRITY_*`, `REVCORE_SECURITY_TEST_REPORT.md` |
| 2026-11-16 | Build 161126 resmi, perbaikan Gate 10, sentinel dan traceability | `dist/RevCore build 161126.apk`, `REVCORE_OFFICIAL_RELEASE_BUILD_REPORT_161126.md`, `REVCORE_SENTINEL_REQUIREMENTS_TRACEABILITY.md` |
| 2026-11-20 | Working copy Live Data + Keyless | `work/REVCORE_KEYLESS_20261120/RevCore_work_live_keyless_20261120` |
| 2026-11-22 | Rollback source ke 161126, sepuluh pekerjaan rebuild, empat koreksi absolut, build 221126 | `03_GIT_HISTORY/`, `delivery/rebuild_161126_20261122/`, `dist/RevCore build 221126.apk` |
| 2026-11-23 | Backup migrasi AI disusun | Paket `REVCORE_AI_MIGRATION_BACKUP/` |

Catatan: tanggal adalah tanggal laporan atau tanggal artefak, bukan klaim tanggal
penulisan setiap baris source. Banyak source berasal dari ekspor ZIP sehingga
timestamp internalnya dinormalisasi (1981) dan tidak dapat dipakai sebagai bukti
kronologi.

---

## 1. Asal-usul dan tujuan RevCore

RevCore berawal dari aplikasi Android untuk berkomunikasi dengan ECU Honda
(Keihin dan Shindengen) melalui adapter USB–K-Line. Jejak asal yang masih ada:

- `README.md` baseline masih berisi instruksi proyek **Google AI Studio** dan
  tautan aplikasi `984b7af0-1371-4f0e-8c9d-b9224bd24445`.
- Folder `.aistudio/artifacts/brain/984b7af0-.../implementation_plan.md` memuat
  rencana awal "Audit & Perbaikan Live Telemetry & Clear DTC / MIL Status".
- Rencana itu sudah menyebut dua masalah inti yang kemudian menjadi tema besar
  proyek: data sensor yang semu dan perintah Clear DTC yang salah.

Tujuan yang terbentuk dari dokumen-dokumen tersebut:

1. Menampilkan Live Data yang benar-benar berasal dari ECU, bukan angka nol atau
   nilai karangan.
2. Membaca dan menghapus DTC secara jujur, tanpa mengklaim lampu MIL fisik padam
   bila status ECU tidak terbukti.
3. Menyediakan identifikasi kendaraan/ECU, Keyless ID, serta Tuning/Remap dengan
   batas bukti yang jelas.
4. Menjadi alat belajar dan penelitian, bukan alat yang menyembunyikan batas
   pengetahuan.

Filosofi resmi yang kemudian dicantumkan di aplikasi:

> "RevCore — Jembatan untuk belajar, bukan pagar untuk membatasi pengetahuan."

## 2. Identitas Otoy sebagai pemilik dan pencipta

- Nama pemilik dan pencipta: **Otoy Sullivan** (`otoy`).
- Nama aplikasi: **RevCore**.
- Package: **`com.revcore.android`**.
- Kredit pencipta di aplikasi: **"otoy"**.
- Peran AI/executor: alat analisis dan pelaksana teknis; keputusan akhir tetap
  milik Otoy.

Identitas ini tidak boleh diubah oleh AI penerus. Perubahan package, nama,
kredit, branding, atau tema UI memerlukan perintah langsung Otoy.

## 3. Perkembangan arsitektur aplikasi

Perkembangan yang tercatat:

- Awalnya aplikasi AI Studio dengan UI dan logika komunikasi yang masih
  bercampur.
- Audit keamanan 2026-11-14 menemukan build "release" masih memakai debug
  keystore dan `debuggable`; ini memicu pemisahan signing resmi dan aturan build.
- Arsitektur kemudian dipisahkan menjadi:
  - UI Jetpack Compose (`ui/`, `ui/screens/`, `ui/keyless/`, `ui/livedata/`).
  - `MainViewModel` sebagai koordinator startup, sesi, dan operasi.
  - `core/protocol` dan `core/comm` untuk protokol Honda, Live Data, dan sesi.
  - `core/usb`, `core/ftdi`, `core/kline` untuk transport.
  - `core/keyless`, `core/remap`, `data/maps` untuk fitur kendaraan.
  - `security/` untuk Gate 1–10, trust record, dan installation marker.
- Build menggunakan Kotlin, Compose, compileSdk/targetSdk 36, minSdk 26, JVM 21,
  R8 minify + resource shrinking.
- Folder kerja 20–22 November **bukan repositori Git**; karena itu bukti
  perubahan di sana berupa snapshot, laporan, dan hash file, bukan riwayat commit.
  Repositori Git `rebuild-161126` baru dibuat 2026-11-22 untuk merekam baseline
  dan rollback.

Bukti struktur: `08_ARCHITECTURE_AND_FEATURES/APP_ARCHITECTURE.md`.

## 4. Sejarah analisis firmware dan reverse engineering

- `libecuwb_ui.so` dianalisis untuk memahami protokol native. Hasilnya dipakai
  membangun lapisan `core/comm` pengganti, dengan salinan legacy untuk rollback.
- `firmware.elf` (SHA-256
  `fe0704fd322d2c981ab029faf63d7463a4d7521cc912e18fa411252e286b65e1`)
  didekompilasi dengan Ghidra 12.2.
- Rantai Keyless: `Immo_reset` (`FUN_400e8f20`) → `FUN_400e51b8` →
  `FUN_400e4d78` (Keihin) / `FUN_400e4fc4` (Shindengen).
- Log dekompilasi disimpan di
  `REVCORE_REFERENCE_AUDIT_20261120/evidence/ghidra_reset_decomp.log`.
- Analisis MAP dan BIN dilakukan dari referensi `refs/1234/` dan `refs/wrt/`,
  termasuk format MAP, parser BIN, dan pertanyaan yang belum terjawab.
- Laporan FlashPro forensik dan rencana perubahan RevCore ada di
  `docs/flashpro/FLASHPRO_REVERSE_ENGINEERING_REPORT.md`.

Batas bukti: frame Keyless dan Remap berstatus
`RECONSTRUCTED_FROM_BINARY`/`PROTOCOL_ANALYZED`. Tidak ada klaim bahwa urutan
frame tersebut sudah diverifikasi pada ECU fisik.

## 5. Perkembangan database kendaraan

- Database awal diperkaya dari APK/referensi Honda.
- BikeStudio diintegrasikan sebagai suplemen non-destruktif (2026-11-05).
- NO.3 (2026-11-08) mengintegrasikan database ECM Indonesia 117 record tanpa
  menghapus data lama.
- Identitas kendaraan FlashPro digabung ke database RevCore.
- Pipeline identifikasi ECU diperbaiki agar ID kanonik dipisahkan dari ID mentah,
  dengan penanganan ambiguitas dan status kompatibilitas.
- Aset database saat ini berada di `app/src/main/assets/ecm_database.json` dan
  termasuk tiga aset yang dilindungi Gate 10.

Database adalah data referensi. Kecocokan database bukan bukti bahwa kendaraan
fisik sudah diuji.

## 6. Perkembangan MAP dan Tuning

- 2026-11-08: katalog MAP, dekripsi, parser, tampilan, dan editor aman (NO.4)
  diimplementasikan. Flashing tetap diblokir.
- Pustaka MAP orisinal diintegrasikan dan diuji; 162 berkas `.map` berada di
  `app/src/main/assets/revcore_maps/` setelah pembersihan identitas.
- 2026-11-11–13: Tuning Studio disatukan, editor diberi undo/redo, lalu UI Remap
  didesain ulang.
- 2026-11-14: redesign Remap di-rollback secara selektif karena keluar dari
  ruang lingkup yang disetujui; rollback mengembalikan file ke snapshot sebelum
  2026-11-13 22:29 WIB.
- 2026-11-22: Tuning disusun ulang mengikuti alur FlashPro (kendaraan/ECM → MAP →
  BIN/remap), tetapi tetap memakai identitas visual RevCore. Pemilihan dan ekspor
  MAP dipertahankan.
- Write BIN tidak diklaim berhasil. `RemapWorkflow` menetapkan
  `WRITE_HARDWARE_VALIDATED = false`; protokol tulis berstatus rekonstruksi dan
  belum diuji pada ECU.

## 7. Perkembangan komunikasi USB, FTDI, dan K-Line

- Transport utama: adapter FTDI FT232R melalui USB-OTG, K-Line Honda 10400 bps.
- Generasi awal masih memakai tombol hubung manual; kemudian OTG auto-detect
  ditambahkan.
- Masalah yang pernah terjadi dan diperbaiki:
  - Inisialisasi K-Line gagal atau kehilangan fallback.
  - Frame ACK pendek diperlakukan sebagai data sensor.
  - Echo adapter dianggap sebagai jawaban ECU.
  - Port dianggap terbuka padahal perangkat USB sudah hilang.
  - Balasan terlambat masuk ke sesi berikutnya.
- Perbaikan 2026-11-22:
  - `ProtocolManager` menjadi pemilik tunggal jalur kawat dan lease eksklusif
    untuk Keyless/Remap.
  - Setiap transaksi memeriksa transport yang masih hidup.
  - Tiga kegagalan I/O berturut-turut menandai transport tertutup.
  - Reset ECU yang terkonfirmasi membatalkan status identifikasi/Live/DTC lama.
- Kestabilan pada kendaraan fisik tetap **OWNER VALIDATION PENDING**.

## 8. Perkembangan Keyless ID

- 2026-11-20: audit FlashPro Keyless ID dan source RevCore; analisis FTDI K-Line;
  delapan dokumen audit disusun.
- Read Keyless menggunakan frame hasil dekompilasi:
  - Keihin: `82 82 10 06`, timeout 400 ms, balasan ≥ 12 byte, data di +10/+11.
  - Shindengen: `91 91 07 40`, timeout 300 ms, balasan ≥ 13 byte, data di
    +0x0B/+0x0C.
  - 256 alamat dibaca.
- RESET menulis nol ke area registrasi:
  - Keihin: indeks `0x46..0x49`, frame `82 82 14 08 ...`, checksum
    `-(page + 0x20 + low + high)`, jeda 120 ms, sukses bila 0 dari 4 gagal.
  - Shindengen: alamat `0x00..0xAF`, frame `91 91 09 41 ...`, checksum
    `-0x4A - address`, jeda 5 ms, sukses bila < 10 dari 176 gagal.
- RENEW kemudian diputuskan memakai urutan zero-write yang sama dengan RESET,
  bukan menulis material kunci baru. Keputusan ini tercatat di commit
  `329405a` (2026-11-22 09:59) dan tetap ada di working copy.
- **Dokumentasi lama tidak seragam:** `delivery/change_summary.md`,
  `delivery/unresolved_items.md`, dan `delivery/protocol_evidence_matrix.md`
  masih menyebut `RENEW_SUPPORTED=false`. Source aktual memiliki
  `KeylessRenewPlan.RENEW_SUPPORTED = true`. Perbedaan ini dicatat sebagai
  drift dokumentasi di `10_KNOWN_ISSUES/UNRESOLVED_AND_LIMITS.md`; dokumen lama
  tidak dihapus.
- ACK saja bukan bukti keberhasilan; engine membaca ulang target dan hanya
  menyatakan `SUCCESS` bila hasil baca-balik sesuai. Keihin yang menulis pola
  literal dapat menghasilkan `PARTIAL`.
- Pengujian pada ECU dan smartkey fisik belum dilakukan.

## 9. Sejarah Gate 10

- 2026-11-14: audit Phase 0 menemukan APK "release" 141126 masih debug-signed,
  `debuggable`, dan memakai `debug.keystore`. Temuan ini menjadi dasar sistem
  keamanan resmi.
- Gate 1–10 dirancang sebagai pemeriksaan berlapis:
  - GateId tetap 1–10; `FINAL_TRUST = 8`.
  - Urutan eksekusi staged: 1,2,3,4,5,6,7,9,10,8.
  - Gate 8 hanya lulus bila sembilan gate lain PASS.
- Gate 10 memeriksa inventaris aset APK terhadap `integrity_policy.json`,
  dengan pengecualian tepat tiga berkas: policy itu sendiri, UI manifest, dan
  `dexopt/`.
- 2026-11-15: UI/DEX integrity, manifest bertanda tangan, trust record
  `[berkas trust eksternal - tidak dipublikasikan]`, dan installation marker diimplementasikan.
- Build 151126 dan 161126 memakai prosedur rilis resmi dengan sertifikat
  `c649cb4f…18d098`.
- 2026-11-16: perbaikan Gate 10 untuk positif palsu inventaris aset; build
  161126 menjadi baseline.
- 2026-11-22: rename aset `flashpro_*` → `revcore_*` mengharuskan regenerasi
  policy. Hasil verifikasi: 165 path policy = 165 aset APK, 0 `flashpro`.
- Gate 10 tidak dihapus, tidak dilemahkan, dan tidak diberi bypass baru.

Rincian mekanisme: `04_GATE10_SECURITY/GATE10_EXPLAINED.md`.

## 10. Keputusan rollback ke Build 161126

Pada 2026-11-22, pekerjaan 20–22 November sudah berkembang jauh dari build
resmi terakhir. Working copy berisi fitur baru, tetapi juga berbagai perubahan
UI dan protokol yang belum diterima. Pemilik memutuskan:

1. Build **161126** dijadikan baseline rebuild yang stabil dan resmi.
2. Source aplikasi di-rollback ke kondisi 161126 melalui commit
   `656bed2 rollback: restore app source to Build 161126`.
3. Rollback didokumentasikan dalam commit
   `1cc9120 docs(rebuild): record the verified Build 161126 baseline and research archive`.
4. Pekerjaan yang lebih baru **tidak dihapus dari sejarah**. Build 221126,
   laporan 20–22 November, dan seluruh working tree tetap disimpan.
5. Sepuluh pekerjaan rebuild kemudian dijalankan di atas baseline 161126, di
   branch `rebuild-161126`.

Urutan commit yang tersedia (repo 2026-11-22):

```text
7905a4b baseline work copy LM-01
e9492e9 remap read-write-verify
3c730f7 H-Diag DTC probe + stale-session guard
329405a Keyless RENEW zero-write
eaa9117 discard late replies
430cb6a adapter JSON-line protocol
e761002 docs official release 221126 (master)
656bed2 rollback to Build 161126
1cc9120 docs baseline + research archive (rebuild-161126)
```

Repo ini tidak memiliki remote. Riwayat Git sebelum 2026-11-22 tidak tersedia
sebagai commit; buktinya adalah dokumen dan snapshot. Rincian:
`03_GIT_HISTORY/README.md`.

## 11. Sepuluh pekerjaan rebuild

Sepuluh pekerjaan yang diminta setelah rollback:

1. Unified Live Data dan Actuator dalam satu tabel tanpa Role Table.
2. Prakiraan AFR dari formula penelitian.
3. Log TXT, berbagi file, Buka Lokasi, dan event log.
4. Sistem Keyless ID lengkap.
5. Tuning dan Remap dengan FTDI.
6. Protocol Manager dan kestabilan USB.
7. Layar tetap menyala ketika RevCore aktif.
8. Gate 10, jeda 2 detik, animasi loading, lalu menu utama.
9. Susunan UI DTC.
10. Audit dan pembersihan identitas internal RevCore tanpa merusak identitas
    eksternal yang diperlukan.

Status lengkap per pekerjaan (tujuan, source, implementasi, test, bukti,
implementasi, validasi kendaraan, kekurangan) ada di
`07_PROJECT_HISTORY/REBUILD_WORKS_1_10.md`.

## 12. Koreksi dan keputusan terbaru

Empat koreksi UI yang ditetapkan Otoy pada 2026-11-22 dan berlaku sebagai
aturan aktif:

1. **Startup** — Gate 10/trust valid → jeda 2 detik → animasi loading → menu.
   Loading wajib muncul setiap startup; trust record boleh melewati konfirmasi,
   tidak boleh melewati loading.
2. **Live Data** — jangan tampilkan tabel kosong/pesan debug sebelum terhubung;
   hanya parameter valid yang diterima; parameter tambahan muncul otomatis saat
   mesin hidup; parameter lama dipertahankan; AFR hanya setelah O2 valid; jangan
   mengganti sistem polling; jangan mengarang angka.
3. **Keyless ID** — urutan 1 Kendaraan, 2 ECM (Keihin/Shindengen), 3 Baca,
   4 Reset, 5 Renew; Reset/Renew aktif hanya setelah baca berhasil; popup ID
   dapat disalin; protokol mengikuti source riset; ACK bukan bukti.
4. **Tuning** — ikuti pola tata letak FlashPro sebagai referensi, tetap beridentitas
   RevCore; jangan ubah tema sepihak; jangan menempel fitur tanpa integrasi;
   jangan hapus pemilihan/ekspor MAP; jangan klaim Write BIN berhasil.

Teks lengkap dan status implementasinya:
`07_PROJECT_HISTORY/UI_CORRECTIONS_20261122.md`.

Keputusan terbaru lain:

- APK resmi 221126 ditandatangani melalui dialog Owner dan diverifikasi:
  39.841.086 byte, SHA-256
  `cb5892a3b956c566e2f9cb3f0ae77f84488f1cd4902803b0ca756298720ba556`.
- Unit test current: 55 suite / 457 test (laporan koreksi); baseline 161126:
  41 suite / 353 test.
- Build dan test tidak sama dengan validasi ECU fisik.

## 13. Masalah yang pernah terjadi dan penyebab yang diketahui

| Masalah | Penyebab yang diketahui | Status |
|---|---|---|
| Kode hantu `MIL#193-128` | Parser membaca header `C1 80` sebagai kode DTC; Clear memakai service yang salah | Diperbaiki; bukti di laporan one-pass |
| Live Data menampilkan nol/nilai semu | Frame ACK pendek dianggap data; nilai tidak tersedia diwakili 0 | Diperbaiki bertahap; validasi per-field |
| Data hilang saat tabel berganti | Setiap frame mengganti seluruh peta status | Retensi `STALE` ditambahkan 2026-11-22 |
| Port USB "terbuka" padahal hilang | Tidak ada pemeriksaan transport hidup | `requireLiveTransport`, degradasi 3 kegagalan I/O |
| Balasan sesi lama masuk ke sesi baru | Tidak ada penanda sesi pada antrean balasan | Commit `eaa9117` |
| Loading dilewati pada jalur trust/gagal | Beberapa jalur memanggil inisialisasi/gagal langsung | Hold 2 detik disatukan 2026-11-22 |
| Gate 10 positif palsu | Inventaris APK mencakup berkas yang dikecualikan secara keliru | Diperbaiki build 161126 |
| Release 141126 debug-signed | `signingConfig release` memakai `debug.keystore` | Diganti prosedur resmi 151126/161126 |
| ACK Keyless dianggap sukses | Tidak ada baca-balik | Read-back wajib; ACK bukan bukti |
| Identitas `flashpro` tersisa | Nama paket, aset, label, dan ID profil | 185 rename 2026-11-22; identifier eksternal dipertahankan |
| Status RENEW tidak konsisten | Keputusan berubah setelah dokumen delivery ditulis | Source `true`; dokumen lama `false`; dicatat sebagai drift |
| Write BIN berisiko diklaim sukses | Protokol belum diuji pada ECU | Ditahan; `WRITE_HARDWARE_VALIDATED=false` |

## 14. Pekerjaan yang belum selesai

- Validasi perangkat fisik: Poco X3 NFC (surya, Android 12/14) + adapter FTDI +
  kendaraan. Live Data, Keyless, DTC, Gate 10 runtime, dan keep-screen belum
  diuji di perangkat oleh Otoy.
- Keyless RENEW/RESET pada ECU fisik; interpretasi separuh payload nol Keihin.
- Remap/Flash: validasi ECU, offset checksum per profil, perilaku 62.500 baud,
  geometri halaman penuh, recovery power-loss.
- Skala O2 narrowband (nilai 3,765 V) dan AFR terukur dengan wideband.
- MD 0x79, provenance tabel 10, dan pemetaan H-Diag.
- Restore trust record/installation marker belum diuji dari backup.
- Uji restore APK/keystore belum dilakukan (hanya verifikasi hash).
- Dokumen delivery lama perlu diselaraskan dengan status RENEW terbaru.
- Penerimaan akhir dan keputusan migrasi tetap milik Otoy.

---

## Dokumen lanjutan yang harus dibaca

1. `00_READ_FIRST/AI_MIGRATION_START_HERE.md`
2. `00_READ_FIRST/OWNER_AUTHORITY_AND_RULES.md`
3. `04_GATE10_SECURITY/GATE10_EXPLAINED.md`
4. `05_PRIVATE_KEY_INVENTORY/KEY_INVENTORY.md`
5. `07_PROJECT_HISTORY/REBUILD_WORKS_1_10.md`
6. `08_ARCHITECTURE_AND_FEATURES/APP_ARCHITECTURE.md`
7. `10_KNOWN_ISSUES/UNRESOLVED_AND_LIMITS.md`

**Keputusan akhir dan persetujuan migrasi tetap milik Otoy.**
