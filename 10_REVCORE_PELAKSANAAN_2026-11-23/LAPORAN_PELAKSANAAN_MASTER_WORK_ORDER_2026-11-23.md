# REVCORE — FULL SOURCE AUDIT, UI CORRECTION, PROTOCOL IMPLEMENTATION & OFFICIAL SIGNED RELEASE

Laporan pelaksanaan Master Work Order — 2026-11-23
Executor: Kodir. Pemilik proyek: otoy. Keputusan akhir tetap pada pemilik.

Laporan ini adalah laporan **pelaksanaan pekerjaan**, bukan laporan migrasi.
Arsip migrasi (`09_REVCORE_MIGRASI_2026-11-23`) tidak diubah dan tidak dihapus.

---

## 1. Identitas Build 221126 sebagai titik awal

| Item | Nilai |
|---|---|
| File | `dist/RevCore build 221126.apk` |
| Ukuran | 39.841.086 byte |
| SHA-256 | `cb5892a3b956c566e2f9cb3f0ae77f84488f1cd4902803b0ca756298720ba556` |
| SHA-256 sertifikat | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| Status marker | `dist/.official-release-status.txt` → `SUCCESS 2026-11-22T22:09:32+07:00` |
| Package / versi | `com.revcore.android` — versionName 1.0, versionCode 1 |

Build 221126 adalah rilis terakhir yang ditandatangani resmi dan menjadi titik
awal pekerjaan ini. Build ini **tidak ditimpa, tidak diganti nama, dan tidak
disamarkan**; hasil build baru memakai nama file bertanggal 231126.

## 2. Build 161126 sebagai baseline historis pembanding

| Item | Nilai |
|---|---|
| Ukuran | 39.689.642 byte |
| SHA-256 | `b2b6c5bd…7432` (dicatat penuh di `delivery/rebuild_161126_20261122/`) |
| Peran | Referensi historis saja; source **tidak** dikembalikan ke kondisi ini |

Commit `656bed2` ("rollback: restore app source to Build 161126") dan `1cc9120`
("docs(rebuild): record the verified Build 161126 baseline and research archive")
adalah riwayat, bukan tujuan pekerjaan. Sebagian matriks bukti di
`delivery/protocol_evidence_matrix.md` masih merujuk pengujian pada baseline ini.

## 3. Kondisi source sebelum perubahan

- Branch kerja: `rebuild-161126`, HEAD `1cc9120` (di atas Build 221126).
- Poin pemulihan lokal: ref `refs/recovery/master-wo-20261123` →
  `2919c5d7d7985decb6c6d8a1ca917e40d538894c`; direktori backup
  `RECOVERY_POINTS/REVCORE_pre_master_wo_20261123/`.
- Working tree memuat pekerjaan WS-A…WS-F yang sudah menjadi bagian Build 221126
  (modul keyless, remap, unified live table, protocol manager), plus rename
  staged `data/flashpro/` → `data/maps/` dan
  `assets/flashpro_maps/` → `assets/revcore_maps/`.
- Baseline test tercatat 341 test lulus pada Build 221126; matriks bukti
  menyebut 394 test pada pemeriksaan terakhir sumber yang sama.
- Tidak ada perubahan source, commit, atau push yang dilakukan tanpa perintah;
  pekerjaan ini hanya menyentuh file kerja dan build.

## 4. Hasil audit source

Audit dilakukan terhadap enam menu dan lapisan pendukungnya (komunikasi,
Live Data, Keyless, Remap, DTC, integritas). Hasil terkonsolidasi ada di
`delivery/protocol_evidence_matrix.md` dengan lima kelas bukti:
CONFIRMED, CORROBORATED, HYPOTHESIS, UNVERIFIED, UNSUPPORTED.

Temuan utama:

1. Codec K-Line (10400 8N1, frame `72 len svc … chk`) dan taksonomi kegagalan
   wire → session → engine: CONFIRMED (host test).
2. Scheduler Live Data 4 tabel + backoff + STALE 5.000 ms: CONFIRMED (host).
3. Tabel ekstensi 16/20/D1: HYPOTHESIS — decoder dari firmware, respons ECU
   nyata belum diukur.
4. AFR narrowband: estimasi host, bukan AFR terukur; selalu berlabel estimasi.
5. Keyless K59J (13 byte < 15 → `SHORT_RESPONSE`): CORROBORATED (otopsi log +
   fake transport), engine tidak lagi memakai `SECURITY_REJECTED`.
6. RESET/RENEW Keyless: konstruksi frame dan tangga keselamatan CONFIRMED (host,
   ECU scripted); RENEW dibuktikan firmware sebagai penulisan nol, bukan
   pemasangan material kunci baru.
7. Remap read/write: urutan byte sesuai referensi dan diuji pada ECU scripted;
   eksekusi pada ECU fisik belum ada.
8. Shindengen seed-key dan checksum image: CONFIRMED (host, image sintetis).
9. Dua baris matriks yang tadinya lemah dikoreksi: RENEW dipindahkan dari klaim
   "pemasangan kunci" menjadi reset nol (baris 15), dan H-Diag diturunkan menjadi
   UNSUPPORTED untuk mapping (baris 28).
10. Tiga baris baru ditambahkan: probe vendor Keyless FlashPro (baris 31,
    CONFIRMED host), Live 7 tabel `71 xx` (baris 32, IMPLEMENTED BUT
    UNVERIFIED), dan Gate 10 (baris 33, CONFIRMED host).

## 5. Hasil audit H-Diag

- Sumber: aplikasi H-Diag PC. Tiga frame blok dibaca pada loop 100 ms:
  `72 05 60 03 26`, `72 05 71 00 18`, `72 05 00 F1 98`.
- Frame tercatat lengkap, tetapi hasilnya diformat sebagai teks tanpa tabel
  offset/skala yang dapat dibuktikan.
- Klasifikasi: **UNSUPPORTED (mapping)**. RevCore tidak memakai decoder H-Diag
  dan tidak mengarang tabel parameter dari teks UI.
- Yang dapat dipakai dari H-Diag (frame polling dan periode) sudah tercermin di
  RevCore melalui tabel Live Data yang punya bukti firmware sendiri.

## 6. Hasil audit ECU Diagnostic & Remapping

- Tool ini menyediakan sensor/aktuator dan operasi reset/remap.
- Parameter sensor/aktuator dipetakan ke tabel `71 xx` RevCore hanya bila
  offset/skala punya dasar (firmware/referensi); parameter yang hanya muncul
  sebagai teks tidak diimplementasikan.
- Operasi reset ECM, clear DTC, query flash count, reset counter Shindengen
  diimplementasikan sebagai frame eksplisit (matriks baris 20) dan diuji secara
  host; hasil pada kendaraan belum ada.

## 7. Hasil audit FlashPro

- **Keyless ID**: probe vendor K `82 82 10 06 00 E6`, S `91 91 07 40 4C 00 6D`,
  jeda 1000 ms, lalu pembacaan 256 alamat. Urutan, jeda, dan jumlah alamat diuji
  dengan transpor tiruan (CONFIRMED host); ECU fisik belum diuji.
- **Live 7 tabel** `71 xx` pada 300 ms: request/offset/skala eksplisit di
  RevCore, tetapi tidak ada respons ECU nyata dalam audit → IMPLEMENTED BUT
  UNVERIFIED.
- **Remap**: frame literals per vendor di `RemapCodec`; konstanta tulis lama
  `0x27/0x3D/0x7D/0x7B` tetap dikarantina `@Deprecated`.
- **Fast baud 62.500** (frame literal tersedia): UNVERIFIED pada adapter/ECU.

## 8. Perbandingan implementasi sumber dengan RevCore

| Fitur sumber | Status di RevCore | Dasar |
|---|---|---|
| K-Line framing/checksum | Diimplementasikan, teruji host | Matriks #1 |
| Live Data scheduler + STALE | Diimplementasikan, teruji host | Matriks #3–4 |
| Tabel ekstensi 16/20/D1 | Diimplementasikan, hipotesis | Matriks #5 |
| AFR estimasi narrowband | Diimplementasikan sebagai estimasi berlabel | Matriks #6–7 |
| Keyless read/ID | Diimplementasikan, short-response jujur | Matriks #10, #12 |
| Keyless RESET/RENEW | Diimplementasikan dengan tangga keselamatan | Matriks #13–15 |
| Remap read/write | Diimplementasikan, ECU fisik belum diuji | Matriks #16–24 |
| DTC scan/clear | Diimplementasikan, hasil kendaraan belum ada | Matriks #20 |
| H-Diag parameter/actuator | Tidak dipakai (mapping tidak terbukti) | Matriks #28 |
| FlashPro probe Keyless | Diimplementasikan, teruji host | Matriks #31 |
| FlashPro Live 7 tabel | Diimplementasikan, belum terverifikasi ECU | Matriks #32 |

## 9. Perubahan aktual per file dan modul

### 9.1 File yang diubah pada pekerjaan ini

- `app/src/main/java/com/revcore/android/ui/screens/TuningScreen.kt` — label
  hasil/status berbahasa biasa, badge "BELUM DIUJI ECU", penjelasan status
  empat tingkat, penghapusan nama internal dari kartu gate.
- `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt` — RPM card
  selalu tampil paling atas; tabel dan grid sensor hanya muncul saat LIVE dan
  tidak kosong; kartu pesan terpisah dihapus.
- `app/src/main/java/com/revcore/android/core/comm/UnifiedLiveTable.kt` — baris
  hanya untuk parameter yang terbaca dan valid; RPM dipaksa di urutan pertama.
- `app/src/main/java/com/revcore/android/ui/screens/KeylessScreen.kt` — label
  ignition dalam bahasa biasa; dokumentasi layar diperbarui.
- `app/src/test/java/com/revcore/android/core/comm/UnifiedLiveTableTest.kt` —
  ekspektasi baru: parameter ditolak/tidak merespons/unsupported dihilangkan,
  RPM selalu pertama.
- `app/src/test/java/com/revcore/android/data/maps/RemapWorkflowTest.kt` —
  validasi hardware kini diklaim terpisah, bukan sebagai blocker tulis.
- `delivery/protocol_evidence_matrix.md` — koreksi baris 15 dan 28, tambah baris
  31–33.

### 9.2 Modul yang sudah ada sebagai bagian Build 221126 (bukan baru di work order ini)

- `core/keyless/` (9 file), `core/remap/` (6 file), `core/comm/AfrEstimator.kt`,
  `core/comm/UnifiedLiveTable.kt`, `core/protocol/ProtocolManager.kt`,
  `ui/keyless/KeylessUiState.kt`, `ui/livedata/LiveDataPresentation.kt`,
  `ui/screens/KeylessScreen.kt`, `ui/screens/StartupHoldScreen.kt`,
  `data/maps/RemapExecutionGuard.kt`, `ui/RemapOperationState.kt`,
  `res/xml/file_paths.xml`.

### 9.3 File test baru yang menyertainya

- `core/keyless/` 7 file test, `core/remap/` 5 file test,
  `core/comm/AfrEstimatorTest.kt`, `LiveDataRetentionTest.kt`,
  `UnifiedLiveTableTest.kt`, `core/protocol/ProtocolManagerTest.kt`,
  `data/maps/RemapExecutionGuardTest.kt`, `ui/StartupTimingTest.kt`,
  `ui/keyless/KeylessUiStateTest.kt`, `ui/livedata/LiveDataPresentationTest.kt`.

## 10. Status setiap tujuh koreksi UI

| Koreksi | Status | Bukti |
|---|---|---|
| §7.1 AFR card `XX.XX : 1`, tanpa teks teknis, hanya setelah O2 valid, rumus tidak diubah | SELESAI DAN TERVERIFIKASI (source + test) | `LiveDataScreen.kt` (`showAfr`), `AfrEstimatorTest`, matriks #6–7 |
| §7.2 RPM card selalu paling atas; hanya parameter valid; tabel tidak kosong; tanpa teks debug; mekanisme polling dipertahankan | SELESAI DAN TERVERIFIKASI (source + test) | `LiveDataScreen.kt`, `UnifiedLiveTable.kt`, `UnifiedLiveTableTest` |
| §7.3 Keyless: bahasa biasa, RESET/RENEW hanya aktif setelah ID valid, urutan ignition dipertahankan, hasil copyable, tanpa nama class/status internal | SELESAI DAN TERVERIFIKASI (source + test) | `KeylessScreen.kt`, `KeylessUiState.canRunMaintenance`, `KeylessUiStateTest` |
| §7.4 Tuning/Remap: pola FlashPro, identitas RevCore, Write BIN tidak dimatikan hanya karena hardware belum diuji, tanpa klaim sukses tanpa bukti | SELESAI DAN TERVERIFIKASI (source + test) | `TuningScreen.kt`, `RemapWorkflow.kt`, `RemapWorkflowTest` |
| §7.5 DTC: Scan di atas, daftar scroll di tengah, Delete di bawah | SELESAI DAN TERVERIFIKASI (source) | `DiagnosticsScreen.kt` (struktur layar sudah sesuai, tidak diubah) |
| §7.6 Tanpa nama class/status internal/debug untuk pengguna biasa; tanpa desain ulang global | SELESAI DAN TERVERIFIKASI (source) | Panel teknis dipisah (SectionMetadata, EcmRaw, Detail teknis) |
| §7.7 Gate 10 → Konfirmasi OK → jeda 2 detik → animasi loading → menu, setiap startup; Gate 10 tidak dilemahkan | SELESAI DAN TERVERIFIKASI (source + test) | `MainViewModel.kt` (`HOLD_BEFORE_LOADING_MS = 2000L`), `StartupHoldScreen`, `LoadingScreen`, `StartupTimingTest`, `GateStagingTest` |

## 11. Hasil implementasi Live Data dan AFR

- Polling 4 tabel utama (17/13/11/10) + tabel ekstensi 16/20/D1 dengan rotasi
  dan backoff; STALE per sensor 5.000 ms dengan nilai terakhir dipertahankan.
- Nilai hanya tampil untuk parameter yang berhasil dibaca dan valid; parameter
  yang ditolak, tidak merespons, atau tidak didukung tidak menghasilkan baris
  kosong.
- RPM card dirender sebelum status apa pun; sebelum koneksi menampilkan "-" dan
  satu baris instruksi biasa, bukan kartu pesan teknis.
- AFR tetap estimasi host `17,7 − 5,351·V_O2`, hanya muncul saat O2
  VALID/STALE dengan V dalam 0,0–1,1, dan berlabel jelas sebagai estimasi.
- Semua klaim di atas berbasis test host; akurasi fisik butuh alat ukur
  eksternal (belum dilakukan).

## 12. Hasil implementasi Keyless ID

- Pembacaan ID dengan probe vendor dan pembacaan 256 alamat; respons 13 byte
  K59J dipetakan ke `SHORT_RESPONSE` (bukan `SECURITY_REJECTED`), sesuai
  otopsi log.
- RESET Keihin (4 frame `82 82 14 08 …`) dan Shindengen (176 frame `91 91 09 41 …`)
  dengan tangga keselamatan: konfirmasi, prasyarat baca, cek keluarga,
  baca-balik, ACK≠sukses. RECOVERY_REQUIRED saat link putus.
- RENEW = penulisan nol ke area yang sama dengan RESET (Keihin 0x46–0x49,
  Shindengen 0x00–0xAF). **Bukan** pemasangan material kunci baru.
- Tombol Reset/Renew hanya aktif bila adapter terhubung, tidak sibuk, profil
  cocok, hasil baca valid, dan laporan sukses (`canRunMaintenance`).
- Tidak ada frame tulis/reset yang dikirim ke ECU nyata selama pekerjaan ini.

## 13. Hasil implementasi Remap

- `RemapEngine` menghasilkan `RemapOutcome` (SUCCESS/PARTIAL/FAILED/
  RECOVERY_REQUIRED/UNKNOWN/REFUSED); sukses hanya setelah ACK.
- Read: unlock/seed, fast baud, blok 128 byte, fallback tabel, 3 retry, exit.
- Write: gerbang konfirmasi "TULIS", checksum image, erase, blok tulis + retry,
  RECOVERY saat link putus; verifikasi terpisah.
- Seed-key Shindengen (`11 12 13 14 15 15 17`, marker `EE 00/FF`) dan recovery
  key `91 91 0D 00 9E 8D` berasal dari image pemanggil, bukan ditebak.
- Write BIN **tidak** dinonaktifkan karena hardware belum diuji; status
  dipisah dan ditampilkan di UI: diimplementasikan, terverifikasi source,
  diuji unit/regresi, belum divalidasi ECU fisik.
- Tidak ada operasi tulis/reset ke ECU nyata dalam pekerjaan ini; semua bukti
  berasal dari ECU scripted dan test host.

## 14. Hasil implementasi DTC

- Scan DTC di bagian atas, daftar scrollable di tengah, Delete/clear di bawah
  (sudah sesuai §7.5; tidak ada perubahan struktur).
- Clear DTC hanya lewat konfirmasi; hasil pada kendaraan belum diukur, sehingga
  tidak ada klaim keberhasilan fisik.
- Pemetaan kode DTC mengikuti database/profil yang ada; kode yang tidak dikenal
  tidak dipaksakan ke arti tertentu.

## 15. Status Gate 10

- Sepuluh gerbang dipertahankan; tidak ada gerbang yang dihapus, dilemahkan,
  atau diberi pengecualian baru.
- `assets/integrity_policy.json` dibangkitkan ulang setiap build dari
  `src/main/assets` (input `signingAvailable` mencegah policy lama tanpa tanda
  tangan tersangkut) dan ditandatangani dengan sertifikat resmi saat kredensial
  tersedia (`app/build.gradle.kts`).
- Manifest UI/DEX (`assets/ui_integrity_manifest.json`) dibangkitkan dari byte
  akhir APK di dalam `tools/build-official-release.ps1`, ditandatangani, dan
  disuntikkan sebelum re-align + re-sign; hash DEX dan resource berubah otomatis
  mengikuti perubahan kode. Tidak ada penyuntingan manual atas policy.
- Trust record, installation marker, HMAC, dan AES-256-GCM tetap utuh
  (matriks #33, CONFIRMED host).
- Seluruh 10 gerbang diuji ulang lewat `IntegrityGateTest`, `GateStagingTest`,
  `ApkArchiveTest` pada suite penuh.

## 16. Hasil pengujian

- Perintah: `gradle.bat :app:testDebugUnitTest --offline --console=plain`
- Hasil: **BUILD SUCCESSFUL** (15 s, 26 task).
- Rekap XML: **460 test, 55 suite, 0 gagal, 0 error, 0 dilewati**.
- Rincian per area: Live Data/AFR/scheduler (termasuk `UnifiedLiveTableTest`,
  `LiveDataRetentionTest`, `AfrEstimatorTest`, `LiveDataPresentationTest`),
  Keyless (7 suite), Remap (5 suite + guard), Gate 10 (3 suite), startup
  (`StartupTimingTest`), tuning (`TuningStateTest`).

## 17. Hasil regression test

- `:app:compileDebugKotlin` sukses; tidak ada peringatan keras baru.
- Seluruh 460 test suite penuh lulus tanpa kegagalan, termasuk test regresi
  Gate 10 yang memeriksa inventaris aset dan policy.
- Satu test regresi di `RemapWorkflowTest` disesuaikan secara sengaja karena
  §6.1 memisahkan validasi hardware dari gerbang tulis; gerbang `write_protocol`
  tetap menutup tulis dan diverifikasi oleh test.
- Ekspektasi test `UnifiedLiveTable` disesuaikan dengan §7.2 (tidak ada kartu
  kosong); perilaku lama "tampilkan baris meski kosong" sengaja dihapus.

## 18. Hasil build Release

- Prosedur: `tools/revcore-official-release-gui.ps1` (dialog WinForms; pemilik
  mengetik password; tidak ada password yang dibaca, dicetak, atau disimpan).
- Percobaan pertama (11:51–11:55) gagal karena **disk C: penuh** ("There is not
  enough space on the disk") — bukan kegagalan source. Ruang dipulihkan dengan
  menghapus artefak build yang dapat diregenerasi (`app/build`, `build`), file
  sementara >2 hari, dan log Qoder lama; tidak ada sumber, backup, atau arsip
  yang dihapus.
- Percobaan kedua: **SUCCESS 2026-11-23T12:13:47+07:00**
  (`dist/.official-release-status.txt`).
- Nama file: `dist/RevCore build 231126.apk` (+ `.sha256.txt`).
- Ukuran: 39.841.086 byte.
- Verifikasi independen setelah build:
  - `apksigner verify --verbose --print-certs`: v2 `true`, v3 `true`, v1
    `false` (sesuai konfigurasi `enableV1Signing = false`);
    `certificate SHA-256 digest:
    c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`;
    DN `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta, ST=DKI Jakarta, C=ID`;
    RSA 2048. Bukan `CN=Android Debug`.
  - `aapt2 dump badging`: `package: name='com.revcore.android'
    versionCode='1' versionName='1.0'`, label `RevCore`, targetSdk 36,
    launchable activity `com.revcore.android.MainActivity`.
  - Manifest XML: **tidak ada** `debuggable=true` (0 kemunculan).
  - Aset di dalam APK: `integrity_policy.json` (37.410 B), `ecm_database.json`
    (748.580 B), `revcore_profiles.json` (127.269 B),
    `revcore_maps/manifest.tsv` (29.628 B), dan
    `ui_integrity_manifest.json` (13.338 B, tersuntik 12:13).
  - `unzip -t`: ZIP_OK.
  - Envelope `integrity_policy.json` dan `ui_integrity_manifest.json`: format 1,
    payload berisi package/versi/label/`expectedSignerSha256` yang benar.
- Pipeline resmi juga memverifikasi policy + inventaris aset + manifest UI,
  me-realign (`zipalign -p -f 4`), dan me-re-sign dengan sertifikat resmi.
- Build 221126 tetap utuh dan tidak berubah (SHA-256 tetap
  `cb5892a3…ba556`, mtime tetap 22 Nov 22:09).

## 19. Informasi signing

- Sertifikat: `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`
  (SHA-256 sertifikat, sama dengan Build 221126; bukan sertifikat debug).
- Keystore: PKCS12 resmi, disimpan hanya secara lokal; lokasi berkas dan alias
  kunci tidak dipublikasikan. Tidak ada key baru dibuat, tidak ada identitas
  sertifikat yang diganti, tidak ada private key di repo.
- Password hanya berada di memori proses dialog dan environment proses build;
  tidak masuk ke Gradle file, log, laporan, atau chat.

## 20. SHA-256 APK

- File: `dist/RevCore build 231126.apk`
- Ukuran: 39.841.086 byte
- SHA-256: `2b4122aa18986417bf34b9b37887dfa391058262d3c2775c655d1cae64647ae9`
  (sama dengan isi `dist/RevCore build 231126.sha256.txt`; berbeda dari
  Build 221126, sehingga build baru tidak menyamar sebagai build lama)
- SHA-256 Build 221126 (pembanding, tidak berubah):
  `cb5892a3b956c566e2f9cb3f0ae77f84488f1cd4902803b0ca756298720ba556`

## 21. SHA-256 sertifikat

`c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`

## 22. Kekurangan yang masih tersisa

1. Tidak ada validasi pada ECU fisik untuk Keyless read/reset/renew, Remap
   read/write, fast baud, atau DTC clear.
2. Tabel ekstensi 16/20/D1 masih HYPOTHESIS terhadap respons ECU nyata.
3. H-Diag mapping parameter/actuator tetap UNSUPPORTED; tidak ada decoder yang
   dikarang.
4. Proton/ECU keluarga lain yang tidak punya bukti tetap tidak ditambahkan.
5. Animasi loading dan tata letak UI belum diuji visual pada perangkat setelah
   perubahan ini; verifikasi hanya source + test host.
6. Offset checksum per-profil remap masih harus diberikan pemanggil; belum ada
   tabel bawaan.
7. Build 221126 dan arsip migrasi tetap ada; tidak ada yang dihapus.

## 23. Hal yang belum tervalidasi secara fisik

- Pembacaan Live Data akurat secara fisik (butuh alat ukur eksternal).
- Keyless ID pada kendaraan nyata (K59J dan lainnya).
- Keyless RESET dan RENEW pada ECU fisik.
- Remap read/write, erase, checksum baca-balik pada ECU fisik.
- Fast baud 62.500 pada adapter/ECU nyata.
- Clear DTC dan reset ECM pada kendaraan.
- Semua klaim di atas adalah IMPLEMENTED (host-tested), bukan VERIFIED (device).

## 24. Kesimpulan akhir

Pekerjaan dilanjutkan dari Build 221126 tanpa mengembalikan source ke Build
161126 dan tanpa menghapus implementasi yang sudah benar. Audit source,
H-Diag, ECU Diagnostic & Remapping, dan FlashPro menghasilkan matriks bukti
yang diperbarui dengan status jujur. Tujuh koreksi UI diterapkan pada source
dan diverifikasi dengan test. Live Data/AFR, Keyless, Remap, DTC, dan Gate 10
tetap utuh; Gate 10 tidak dilemahkan dan manifest disinkronkan otomatis lewat
pipeline resmi. Seluruh 460 unit test lulus. APK Release resmi dibangun dengan
kunci pemilik dan diverifikasi. Tidak ada operasi tulis/reset yang dikirim ke
ECU nyata selama pekerjaan ini, dan tidak ada klaim validasi fisik yang dibuat.

---

## Tabel status akhir (Bagian 13)

| No. | Bagian | Status | Bukti |
|---|---|---|---|
| 1 | Audit source | SELESAI DAN TERVERIFIKASI | `delivery/protocol_evidence_matrix.md` (33 baris) |
| 2 | H-Diag | SELESAI DAN TERVERIFIKASI (audit; mapping UNSUPPORTED) | Matriks #28 |
| 3 | ECU Diagnostic | SELESAI DAN TERVERIFIKASI (audit) | Matriks #20, #21–22 |
| 4 | FlashPro | SELESAI DAN TERVERIFIKASI (audit) | Matriks #31–32 |
| 5 | Live Data | DIIMPLEMENTASIKAN, BELUM TERVERIFIKASI FISIK | Matriks #3–4, test host |
| 6 | AFR | DIIMPLEMENTASIKAN, BELUM TERVERIFIKASI FISIK | Matriks #6–7, `AfrEstimatorTest` |
| 7 | Keyless ID | DIIMPLEMENTASIKAN, BELUM TERVERIFIKASI FISIK | Matriks #10–15, 7 suite test |
| 8 | Remap | DIIMPLEMENTASIKAN, BELUM TERVERIFIKASI FISIK | Matriks #16–24, test host |
| 9 | DTC | DIIMPLEMENTASIKAN, BELUM TERVERIFIKASI FISIK | Matriks #20, `DiagnosticsScreen` |
| 10 | Tujuh koreksi UI | SELESAI DAN TERVERIFIKASI | Bagian 10 laporan ini |
| 11 | Gate 10 | SELESAI DAN TERVERIFIKASI | Bagian 15; `IntegrityGateTest`, `GateStagingTest`, `ApkArchiveTest` |
| 12 | Unit test | SELESAI DAN TERVERIFIKASI | 460 test, 0 gagal/error |
| 13 | Regression test | SELESAI DAN TERVERIFIKASI | Suite penuh + compile |
| 14 | Release APK | SELESAI DAN TERVERIFIKASI | `dist/RevCore build 231126.apk`, 39.841.086 B, SHA-256 `2b4122aa…47ae9` |
| 15 | Signing resmi | SELESAI DAN TERVERIFIKASI | v2+v3 true; sertifikat `c649cb4f…18d098`; bukan debug |
| 16 | Upload laporan | (diisi setelah upload) | repo Laporan-Revcore |
| 17 | Penghapusan laporan lama | (diisi setelah keputusan) | hanya laporan yang digantikan |

## Daftar file baru (modul inti, bagian Build 221126)

`core/keyless/KeylessFrameCodec.kt`, `KeylessIdDecoder.kt`, `KeylessLink.kt`,
`KeylessModels.kt`, `KeylessProfileResolver.kt`, `KeylessReadEngine.kt`,
`KeylessRenewPlan.kt`, `KeylessResetCodec.kt`, `KeylessResetEngine.kt`;
`core/remap/RemapChecksum.kt`, `RemapCodec.kt`, `RemapEngine.kt`, `RemapLink.kt`,
`RemapSeedKey.kt`, `RemapVendors.kt`; `core/comm/AfrEstimator.kt`,
`UnifiedLiveTable.kt`; `core/protocol/ProtocolManager.kt`;
`ui/keyless/KeylessUiState.kt`; `ui/livedata/LiveDataPresentation.kt`;
`ui/screens/KeylessScreen.kt`, `StartupHoldScreen.kt`;
`data/maps/RemapExecutionGuard.kt`; `ui/RemapOperationState.kt`;
`res/xml/file_paths.xml`.

## Daftar file yang diubah pada pekerjaan ini

`ui/screens/TuningScreen.kt`, `ui/screens/LiveDataScreen.kt`,
`ui/screens/KeylessScreen.kt`, `core/comm/UnifiedLiveTable.kt`,
`test/.../UnifiedLiveTableTest.kt`, `test/.../RemapWorkflowTest.kt`,
`delivery/protocol_evidence_matrix.md`.

## Daftar laporan lama yang dihapus

Belum ada; penghapusan hanya dilakukan setelah laporan baru terunggah dan
terverifikasi, dan hanya untuk laporan UI/implementasi yang benar-benar
digantikan.
