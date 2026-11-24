# REVCORE — AUDIT, KOREKSI, IMPLEMENTASI PROTOKOL, DATABASE KENDARAAN, DAN PENYESUAIAN GATE 10

Laporan pelaksanaan Work Order — 2026-11-24
Executor: Kodir. Pemilik proyek: otoy. Keputusan validasi/verifikasi akhir tetap pada pemilik.
Branch kerja: `rebuild-161126`. Paket: `com.revcore.android`, versionName 1.0, versionCode 1 (tidak diubah).

Laporan ini hanya mencatat pekerjaan pada work order 2026-11-24. Laporan dan arsip
sebelumnya tidak diubah atau dihapus.

---

## 1. Ringkasan

- Live Data tidak lagi memakai tabel terpisah; setiap baris yang dijawab ECU menjadi
  kartu sensor, dikelompokkan A Sensor Input / B Digital Input-Switch / C Actuator-Output.
- Log: kartu "Manajer Protokol" dan "12 Tahapan Alur Komunikasi" tidak lagi dirender,
  fungsi internalnya tetap ada; jarak daftar diperbaiki; "Buka Lokasi" memakai intent
  direktori/SAF; lokasi penyimpanan tidak berubah.
- Tombol ON/OFF kanan atas memakai satu jalur koneksi yang sama dengan kartu Dashboard
  (`toggleAdapterConnection()`), termasuk inisialisasi USB nyata saat menyambung ulang.
- DTC 0x3000 dinilai sebagai pasangan mentah ECU (kode 0x30, sub 0x00), bukan artefak
  parser; kode tak terpetakan ditampilkan sebagai "DTC BELUM TERPETAKAN" + kode mentah.
- Keyless RESET/RENEW: judul kartu → status → tombol → progress di bawah status → popup
  hasil; progress tidak pernah 100% sebelum ECU mengonfirmasi.
- Seluruh label/flag "Verified Remap" dihapus dari model, parser, database (489 record,
  978 kunci flag), logika, dan teks UI; tidak diganti dengan klaim kemampuan lain.
- Gate 10 tetap aktif; policy integritas ditandatangani ulang per build dari aset final.
- Unit test: 462 test, 0 gagal, 0 error, 0 dilewati.

## 2. Kondisi source awal

| Item | Nilai |
|---|---|
| Branch / HEAD | `rebuild-161126` / `1cc9120` |
| Build resmi terakhir sebelum pekerjaan | `dist/RevCore build 231126.apk` |
| Ukuran / SHA-256 build 231126 | 39.841.086 byte / `1a23ae8dbf028ef8abccd89d0c94122dc2abe751839b143247c5a5971de16128` |
| Sertifikat resmi (pin) | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| Baseline test sebelum perubahan ini | 461 test lulus |
| Working tree | memuat pekerjaan WS-A…WS-F, rename aset MAP, dan perubahan UI sebelumnya |

Build 231126 dijadikan baseline resmi. Build ini tidak ditimpa; build baru memakai
nama file bertanggal sesuai tanggal build.

## 3. Live Data

- Tabel Live berdiri sendiri dihapus dari UI; data diteruskan ke grid kartu sensor
  yang sudah ada (`ui/screens/LiveDataScreen.kt:321-350`). RPM tetap kartu hero.
- Setiap baris hanya muncul bila ECU benar-benar menjawab field tersebut; status dan
  tabel sumber dicantumkan pada kartu. Tidak ada nilai karangan untuk sensor yang
  tidak didukung ECU.
- Klasifikasi kategori (`core/comm/UnifiedLiveTable.kt`, `categoryOf`):
  - A Sensor Input: TPS, MAP, ECT, IAT, O2, baterai, kecepatan, dan sensor telemetri lain.
  - B Digital Input / Switch: gear state, gear position, mode. Hanya status; tidak ada
    kontrol ON/OFF dan tidak ada brake switch karena belum terbukti.
  - C Actuator-Output: injector pulse width dan ignition advance (nilai yang dikendalikan
    ECU, ditampilkan sebagai status; tidak dapat ditulis dari layar ini).
- Tidak ada perubahan pada frame permintaan atau skala; hanya penyajian.

## 4. Log

- Kartu "Manajer Protokol" dan "12 Tahapan Alur Komunikasi" tidak dipanggil lagi dari
  layar. `ProtocolManagerCard` dan state manajer tetap ada di source
  (`ui/screens/LogScreen.kt:618`) sesuai batas "jangan menghapus fungsi internal".
- Jarak entri daftar diseragamkan menjadi 8 dp (`LogScreen.kt:110`) dan daftar dirender
  dengan `itemsIndexed`.
- "Buka Lokasi" mencoba intent direktori Dokumen (mime direktori, resource/folder, root),
  lalu `ACTION_OPEN_DOCUMENT_TREE` dengan URI awal, dan hanya menampilkan pesan bila
  semua gagal (`LogScreen.kt:513`). `<queries>` untuk intent tersebut ada di
  `AndroidManifest.xml`.
- Lokasi penyimpanan log tidak berubah; setiap entri berbeda tetap disimpan.

## 5. USB ON/OFF

- Tombol kanan atas memanggil `viewModel.toggleAdapterConnection()`
  (`ui/MainAppScaffold.kt:330`, `ui/MainViewModel.kt:1799`), bukan jalur kedua.
- Saat OFF: `disconnectAdapter()`. Saat ON: `refreshAdapters()` yang menghapus latch
  auto-connect/permission, memindai adapter, meminta izin USB, lalu memulai
  auto-connect dan identifikasi melalui pipeline yang sama dengan kartu Dashboard.
- Tidak ada mekanisme koneksi paralel atau status ganda.

## 6. DTC 0x3000

- Sumber: pasangan mentah ECU `0x30 0x00` (kode 0x30, sub 0x00) pada pembacaan DTC;
  bukan artefak parser dan bukan kesalahan pemetaan. Tidak ada deskripsi yang dikarang
  dari petunjuk ketidakstabilan kelistrikan.
- Kode tak terpetakan ditampilkan sebagai "DTC BELUM TERPETAKAN" dengan kode mentah
  (`core/protocol/HondaDtcDatabase.kt:31,37`; `ui/screens/DiagnosticsScreen.kt:329-342`).
- Alur uji yang tersedia: pindai → pemetaan → kode tak dikenal → hapus → pindai ulang →
  kemunculan kembali. Hapus DTC hanya dilaporkan sukses setelah pembacaan ulang
  memverifikasi; "terhapus" tidak berarti "diperbaiki".

## 7. Keyless RESET / RENEW (UI)

- Urutan kartu: judul → baris status ("STATUS: SIAP RESET", "STATUS: SEDANG PROSES",
  "STATUS: SUKSES/GAGAL") → tombol → area progress di bawah status
  (`ui/screens/KeylessScreen.kt:690-760`).
- Bar progress horizontal membulat, selebar kartu, tinggi sama pada kedua kartu,
  beranimasi halus (`animateFloatAsState` 240 ms). Progress hanya menjadi 100% bila
  laporan proses berstatus SUCCESS; selama proses nilainya mengikuti tahap nyata.
- Popup hasil menampilkan "RESET/RENEW KEY ID SUKSES/GAGAL" beserta alasan singkat
  dari laporan (`KeylessScreen.kt:367`).
- Prasyarat, pemeriksaan, otorisasi, dan protokol keyless tidak diubah. Tidak ada
  respons simulasi.

## 8. Audit database kendaraan

- `EcmModel` tidak lagi memiliki `isVerifiedDiagnostic`/`isVerifiedRemap`;
  `EcmDatabase` tidak membacanya; `IdentityMatchQuality.VERIFIED` dihapus dan diganti
  `EXACT` (satu-satunya kecocokan persis dengan record database).
- `assets/ecm_database.json`: 489 record, 978 kunci flag verifikasi dihapus, 11 catatan
  status BikeStudio dibersihkan. Jumlah record, id, partNumber, ecmId, dan seluruh
  data teknis tidak berubah; referensi `docs/ref1234/...` yang bersifat read-only tetap
  utuh dan pin SHA-256-nya tidak berubah.
- Label turunan ikut dibersihkan: teks prasyarat WRT, label RPM reference, pesan tabel
  mapping, status kompatibilitas offline (`STRUCTURE_MATCHES_OFFLINE_PROFILE`),
  pesan identifikasi di ViewModel, dan label baris DatabaseScreen.
- Tidak ada klaim pengganti seperti "sudah teruji", "aman", atau "siap tulis". yang
  tersisa hanyalah pernyataan bukti yang jujur (mis. "belum divalidasi ECU fisik",
  "checksum belum diketahui").
- Sinkronisasi database ↔ source: parser, model, dan aset diuji ulang; test referensi
  Indonesia memastikan setiap record referensi tetap ada dengan field teknis identik.

## 9. Protokol yang dibuka

Tidak ditemukan lock internal yang menutup fitur yang sudah diimplementasikan:

- Jalur tulis Remap tidak lagi digerbangi flag kurasi "remap verified". Gerbangnya kini
  hanya prasyarat teknis (adapter, model teridentifikasi, keluarga vendor, ukuran image,
  lokasi checksum Keihin) plus konfirmasi TULIS yang diketik pengguna
  (`data/maps/RemapExecutionGuard.kt`).
- RESET dan RENEW Keyless tidak diblokir oleh label verifikasi; keduanya berjalan
  setelah prasyarat baca Key ID dan konfirmasi.
- Tidak ada perubahan pada otorisasi, checksum, atau urutan frame saat membuka lock.

## 10. Protokol yang diimplementasikan / diselesaikan

- K-Line 10.400 8N1, frame `72 len svc … chk`, checksum aditif: CONFIRMED (host).
- Taksonomi kegagalan wire → session → engine: CONFIRMED (host).
- Scheduler Live Data 4 tabel + STALE 5.000 ms: CONFIRMED (host).
- Keyless ID read 256 alamat: CONFIRMED (host).
- Keyless RESET: 4 frame Keihin / 176 frame Shindengen, dengan baca-balik dan ladder
  keselamatan: CONFIRMED (host); RENEW menulis nol ke area yang sama (bukan pemasangan
  material kunci baru): `RENEW_SUPPORTED=true`.
- Remap read (unlock/seed, fast baud, blok 128 byte, 3 retry, exit): CONFIRMED (host).
- Remap write/erase host-side (gerbang konfirmasi, checksum image, blok tulis + retry,
  RECOVERY saat link putus): CONFIRMED (host), belum dieksekusi pada ECU fisik.
- Shindengen seed-key dan checksum image Keihin/Shindengen: CONFIRMED (host, sintetis).
- Maintenance (reset ECM, clear DTC, query flash count, reset counter Shindengen):
  CONFIRMED (host).
- Hapus DTC dengan verifikasi baca ulang: CONFIRMED (host).

## 11. Protokol yang belum dapat diimplementasikan dan alasannya

- Tabel ekstensi Live 16/20/D1: HYPOTHESIS; decoder berasal dari firmware, respons ECU
  nyata belum diukur.
- Akurasi fisik Live Data: UNVERIFIED; memerlukan alat ukur eksternal.
- Fast baud 62.500 (frame literal Keihin/Shindengen): UNVERIFIED; perilaku adapter/ECU
  pada baud ini belum diuji.
- Geometri area reset Keihin 0x46–0x49 dan Shindengen 0x00–0xAF: UNVERIFIED; arti payload
  dan geometri halaman penuh belum terbukti.
- Remap write/readback/recovery pada ECU fisik: UNVERIFIED; belum ada pengujian bench/
  kendaraan, sehingga `FlashCapabilities` tetap false dan sukses tidak diklaim.
- H-Diag parameter/actuator: UNSUPPORTED (mapping); tiga frame blok tercatat, tetapi
  tidak ada tabel offset/skala, jadi decoder tidak dibuat.
- Mode 0x5F dan Altitude 0x79: UNVERIFIED; tidak digabung atau ditafsirkan.
- Konstanta tulis 0x27/0x3D/0x7D/0x7B: tetap dikarantina `@Deprecated`; jalur Remap
  memakai frame literal `RemapCodec`.

Rincian kelas bukti ada di `delivery/protocol_evidence_matrix.md` dan
`delivery/PROTOCOL_AUDIT_20261124.md`.

## 12. Gate 10

- Gate 10 tetap berfungsi penuh: 10 gerbang, trust record HMAC, marker instalasi,
  AES-256-GCM. Tidak ada gerbang yang dimatikan, dilewati, atau dilemahkan.
- Policy integritas ditandatangani ulang oleh task Gradle
  `generateIntegrityPolicyRelease` pada setiap build resmi dari `src/main/assets`
  final; manifest UI/DEX dibuat ulang dari APK final oleh skrip rilis.
- Perubahan sah pada aset/database memperbarui baseline hash melalui policy bertanda
  tangan baru, bukan dengan mengecualikan file dari pemeriksaan.
- `integrity_policy.json` dan `ui_integrity_manifest.json` tetap dikecualikan dari
  inventaris isi APK sesuai desain skrip rilis.

## 13. Hasil uji tiap bagian

| Bagian | Hasil | Bukti |
|---|---|---|
| Live Data kartu + kategori | FIXED (host) | Unit test `UnifiedLiveTableTest` termasuk injector/ignition sebagai actuator; compile lulus |
| Log sembunyi kartu + spasi + Buka Lokasi | FIXED (host) | Source review; intent + `<queries>`; tanpa render kartu |
| USB ON/OFF satu jalur | FIXED (host) | `toggleAdapterConnection` → `refreshAdapters`; review alur |
| DTC 0x3000 | FIXED (display/audit) | Kode mentah + label belum terpetakan; tidak ada klaim arti |
| Keyless UI | FIXED (host) | Status-first, progress bertahap, popup hasil; 100% hanya saat SUCCESS |
| Database bebas label verifikasi | FIXED | 489 record, 0 kunci `isVerified`; test referensi lulus |
| Gate 10 | PRESERVED | Task policy bertanda tangan; tidak ada bypass |
| Unit test | 462 lulus | `app/build/test-results/testDebugUnitTest` |

Catatan: FIXED di sini berarti perubahan source/unit test sudah terpasang dan lulus.
Validasi pada kendaraan/ECU fisik ditentukan oleh pemilik (UNVERIFIED sampai laporan
perangkat).

## 14. Masalah tersisa

- Pengujian ECU fisik dan perangkat Android belum dilakukan pada work order ini.
- Tulis/erase Remap dan RESET/RENEW Keyless masih berisiko pada ECU nyata; belum ada
  pengujian bench/kendaraan.
- Kode DTC 0x3000 tetap tanpa deskripsi karena tidak ada sumber terverifikasi.
- Akurasi Live Data dan fast baud memerlukan pengukuran eksternal.
- Gate 10 bergantung pada sertifikat/policy per build; policy build 241126 sudah
  diverifikasi konsisten dengan APK final.
- Disk C: sempat penuh pada percobaan build pertama; ruang dibebaskan dengan
  memindahkan output build dan arsip APK lama ke D:, bukan menghapus bukti.

## 15. Hasil build & uji APK

| Item | Nilai |
|---|---|
| File | `dist/RevCore build 241126.apk` |
| Ukuran | 39.836.990 byte |
| SHA-256 | `d6dec722c675760b80df1017bd5442c3ae9743835da37ea5c5b506431572f0f2` |
| Status marker | `dist/.official-release-status.txt` → `SUCCESS 2026-11-24T14:24:05+07:00` |
| Tanda tangan | v2 + v3 (v1 false), 1 signer |
| Sertifikat SHA-256 | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` (sama dengan pin) |
| Package / versi | `com.revcore.android`, versionName 1.0, versionCode 1, minSdk 26, targetSdk 36 |
| Gate 10 policy | signature SHA256withRSA; 165 aset; 3 protected (ecm_database.json, revcore_maps/manifest.tsv, revcore_profiles.json); 0 missing, 0 mismatch |
| UI/DEX manifest | 20 UI, 1 DEX, 1 struktur; policyDigest cocok; signer cocok |
| Unit test final | 462 test, 0 gagal, 0 error, 0 skipped |

Catatan build: percobaan pertama pada 13:15 gagal karena disk C: penuh (229 MB bebas),
bukan karena source. Output build regenerable dan arsip APK lama dipindahkan ke
`D:\RevCore_offload_20261124\`; build ulang berhasil dan diverifikasi ulang secara
independen (bukan hanya status skrip).

Uji perangkat/ECU fisik belum dilakukan oleh executor; pemilik adalah penguji final.

## 16. Daftar file berubah (work order ini)

Source utama:
- `app/src/main/java/com/revcore/android/core/comm/UnifiedLiveTable.kt`
- `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt`
- `app/src/main/java/com/revcore/android/ui/screens/LogScreen.kt`
- `app/src/main/java/com/revcore/android/ui/screens/DiagnosticsScreen.kt`
- `app/src/main/java/com/revcore/android/core/protocol/HondaDtcDatabase.kt`
- `app/src/main/java/com/revcore/android/ui/screens/KeylessScreen.kt`
- `app/src/main/java/com/revcore/android/ui/MainAppScaffold.kt`
- `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`
- `app/src/main/AndroidManifest.xml`

Database/remap/identitas:
- `app/src/main/assets/ecm_database.json`
- `app/src/main/java/com/revcore/android/data/ecu/EcmModel.kt`
- `app/src/main/java/com/revcore/android/data/ecu/EcmDatabase.kt`
- `app/src/main/java/com/revcore/android/data/ecu/EcuIdentity.kt`
- `app/src/main/java/com/revcore/android/data/maps/RemapWorkflow.kt`
- `app/src/main/java/com/revcore/android/data/maps/MapValidator.kt`
- `app/src/main/java/com/revcore/android/ui/screens/TuningScreen.kt`
- `app/src/main/java/com/revcore/android/ui/screens/DatabaseScreen.kt`
- `app/src/main/java/com/revcore/android/core/protocol/HondaProtocolEngine.kt`
- `app/src/main/java/com/revcore/android/core/protocol/legacy/LegacyHondaProtocolEngine.kt`
- `app/src/main/java/com/revcore/android/core/protocol/ConnectionPipelineStep.kt`

Test:
- `app/src/test/java/com/revcore/android/core/comm/UnifiedLiveTableTest.kt`
- `app/src/test/java/com/revcore/android/data/ecu/EcmIdentityTest.kt`
- `app/src/test/java/com/revcore/android/data/ecu/EcmIdentityMatcherTest.kt`
- `app/src/test/java/com/revcore/android/data/ecu/EcmIndonesianReferenceTest.kt`
- `app/src/test/java/com/revcore/android/data/maps/RemapExecutionGuardTest.kt`
- `app/src/test/java/com/revcore/android/data/maps/BundledMapAssetsTest.kt`
- `app/src/test/java/com/revcore/android/data/maps/MapCatalogFlashEngineTest.kt`
- `app/src/test/java/com/revcore/android/data/maps/MapParserEditorTest.kt`

Dokumen:
- `delivery/WORK_ORDER_20261124_EXECUTION_REPORT.md`
- `delivery/PROTOCOL_AUDIT_20261124.md`

## 17. Bukti pengujian

- `:app:testDebugUnitTest` — BUILD SUCCESSFUL; 462 test, 0 failure, 0 error, 0 skipped
  (dibaca dari `app/build/test-results/testDebugUnitTest/TEST-*.xml`).
- `:app:compileReleaseKotlin` — BUILD SUCCESSFUL (hanya warning yang sudah ada).
- APK 241126: `sha256sum` = status file; `apksigner verify` → v2+v3, cert pin sama.
- Gate 10 final: skrip `verify_gate10.ps1` (salinan bertarget 241126) →
  `ASSET_MISSING=0 ASSET_MISMATCH=0`, `SIGNER_MATCH=True`,
  `UI_POLICY_DIGEST_MATCH=True`, `UI_RESOURCES_AND_DEX_MISMATCH=0`.
- Grep audit: `grep -c isVerified app/src/main/assets/ecm_database.json` → 0;
  tidak ada `EcmModel.isVerified*`, tidak ada `IdentityMatchQuality.VERIFIED`,
  tidak ada label "remap-verified" di UI.
- Bukti build APK resmi dan verifikasi Gate 10 akan dilampirkan pada pembaruan §15
  setelah pemilik menyelesaikan dialog kata sandi.

— Executor: Kodir. Pemilik dan penguji final: otoy.
