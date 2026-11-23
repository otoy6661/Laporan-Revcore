# REVCORE — SEPULUH PEKERJAAN REBUILD

**Baseline:** Build 161126
**Branch:** `rebuild-161126`
**Sumber status:** source di `02_CURRENT_WORKING_STATE/project` dan laporan
`delivery/rebuild_161126_20261122/`
**Tanggal laporan:** 2026-11-23

Catatan penting:

- "Terimplementasi" berarti kode dan unit test tersedia.
- "Tervalidasi perangkat" hanya boleh diklaim setelah Otoy menguji pada
  kendaraan. Tidak ada pekerjaan di bawah ini yang memiliki klaim tersebut.
- "Bukti penelitian" dipisahkan dari "hasil test". Unit test membuktikan perilaku
  host, bukan perilaku ECU.

---

## 1. Unified Live Data dan Actuator dalam satu tabel tanpa Role Table

- **Tujuan:** Menampilkan parameter Live Data dan aktuator dalam satu tabel
  terpadu, dengan jejak sumber per baris, tanpa lapisan Role Table perantara.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/core/comm/UnifiedLiveTable.kt`
  - `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt`
  - `app/src/main/java/com/revcore/android/core/comm/LiveDataProfile.kt`
  - `app/src/main/java/com/revcore/android/core/protocol/HondaProtocolEngine.kt`
- **Implementasi aktual:**
  - `UnifiedLiveRow` menyimpan `sensor`, label, unit, `state`, `sourceTable`,
    `sourceOffset`, `isActuator`, `displayValue`, `ageMs`.
  - `UnifiedLiveTable.build()` hanya memasukkan sensor yang benar-benar ada di
    `parameterStates`/`readings`. Katalog dipakai untuk provenance, bukan daftar
    isian.
  - Nilai hanya ditampilkan untuk state `VALID`/`STALE`; state lain menampilkan
    teks status, bukan nol.
  - Tidak ada Role Table baru dan tidak ada jalur data kedua.
- **Hasil test:** `UnifiedLiveTableTest` 8 test lulus; suite Live Data lain lulus;
  total suite current 55/457 (laporan koreksi) tanpa kegagalan.
- **Bukti penelitian:** Tabel/profil hasil rekonstruksi `HondaLiveDataProfiles`
  (tabel 17, 13, 11, 10 dan ekstensi 16, 20, D1); provenance dari
  `docs/flashpro/` dan audit Live Data.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI pada kendaraan.
- **Kekurangan dan pekerjaan lanjutan:** Perlu validasi per tabel pada ECU fisik;
  tabel 10 masih berstatus provenance belum jelas; aktuator belum diuji.

---

## 2. Prakiraan AFR berdasarkan formula penelitian

- **Tujuan:** Menampilkan perkiraan AFR host-side dari tegangan O2 narrowband,
  selalu berlabel "Prakiraan AFR", tidak pernah sebagai AFR terukur.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/core/comm/AfrEstimator.kt`
  - `app/src/main/java/com/revcore/android/ui/livedata/LiveDataPresentation.kt`
  - `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt`
- **Implementasi aktual:**
  - Formula: `AFR = 17.700001 − 5.351 × V_O2`.
  - Skala byte O2: `O2_COUNTS_PER_VOLT = 50.993698`.
  - Rentang yang diterima: 0,0–1,1 V; di luar itu `Invalid`.
  - O2 hilang → `Unavailable`; O2 STALE tetap disimpan untuk tabel, tetapi AFR
    hanya ditampilkan bila state O2 `VALID`.
  - Label dan caveat selalu menyebut estimasi host, bukan AFR terukur.
- **Hasil test:** `AfrEstimatorTest` 7 test; `LiveDataRetentionTest` test 6
  (`afrEstimateTracksTheO2InputAndStaysUnavailableWithoutIt`).
- **Bukti penelitian:** Decoder referensi `LiveDataEngine.java` case `0x20`;
  O2 tabel 20 byte 4; laporan riset Keyless/Live Data bagian 6–7.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI; belum ada pembanding wideband.
- **Kekurangan dan pekerjaan lanjutan:** Skala O2 narrowband belum terbukti
  (contoh 3,765 V di luar rentang); AFR hanya indikasi, bukan alat tuning.

---

## 3. Log TXT, berbagi file, Buka Lokasi, dan event log

- **Tujuan:** Menyimpan log TXT nyata, membagikan file nyata, membuka lokasi
  dengan jujur, dan mencatat event terstruktur per menu.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/ui/screens/LogScreen.kt`
  - `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`
  - `app/src/main/java/com/revcore/android/data/storage/RevCoreStorage.kt`
  - `res/xml/file_paths.xml`, `AndroidManifest.xml`
- **Implementasi aktual:**
  - `createLogShareFile` menulis `.txt` ke folder share, memeriksa file tidak
    kosong, lalu memakai `FileProvider`.
  - "Buka Lokasi" mencoba beberapa intent folder, lalu folder picker, dan bila
    semua gagal menampilkan pesan kegagalan eksplisit.
  - Event terstruktur mencakup identifikasi, Live Data, DTC, Keyless, Remap,
    USB/link; status sukses hanya ditulis setelah outcome terverifikasi.
- **Hasil test:** Kompilasi dan build debug; `TuningLogStoreTest`; pemeriksaan
  string event. Tidak ada unit test UI untuk pemilih file/folder.
- **Bukti penelitian:** Laporan Task 3 `CHANGE_REPORT.md`; inspeksi `LogScreen`
  dan `MainViewModel`.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI pada perangkat (perilaku file
  manager bergantung pada ROM).
- **Kekurangan dan pekerjaan lanjutan:** Uji share/buka folder pada Poco X3 NFC;
  pastikan tidak ada Toast sukses palsu pada ROM tertentu.

---

## 4. Sistem Keyless ID lengkap

- **Tujuan:** Membaca Key ID, menyediakan RESET dan RENEW dengan urutan UI yang
  disetujui, serta menolak klaim sukses berdasarkan ACK saja.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/core/keyless/*`
  - `app/src/main/java/com/revcore/android/ui/keyless/*`
  - `app/src/main/java/com/revcore/android/ui/screens/KeylessScreen.kt`
  - `MainViewModel.startKeylessRead/startKeylessReset`
- **Implementasi aktual:**
  - Urutan UI: 1 Kendaraan Teridentifikasi, 2 ECM (Keihin/Shindengen),
    3 BACA KEY ID, 4 RESET KEY ID, 5 RENEW KEY ID.
  - RESET/RENEW hanya aktif setelah baca berhasil dan hasilnya masih valid untuk
    profil saat ini.
  - Popup ID menyediakan tombol salin.
  - Read memakai frame firmware; RESET/RENEW memakai urutan zero-write yang sama.
  - Engine membaca ulang target dan hanya menyatakan `SUCCESS` bila hasil
    baca-balik sesuai; ACK bukan bukti.
  - `KeylessRenewPlan.RENEW_SUPPORTED = true` di source current. Dokumen delivery
    lama masih menyebut `false` (drift yang dicatat).
- **Hasil test:** `KeylessFrameCodecTest` 13, `KeylessIdDecoderTest` 7,
  `KeylessProfileResolverTest`, `KeylessReadEngineTest` 6,
  `KeylessRenewPlanTest` 4, `KeylessResetCodecTest` 8,
  `KeylessResetEngineTest` 7, `KeylessUiStateTest` 7.
- **Bukti penelitian:** Dekompilasi `firmware.elf` via Ghidra 12.2; rantai
  `Immo_reset`; log `ghidra_reset_decomp.log`; riset FlashPro Keyless.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI; tidak ada ECU/smartkey fisik.
- **Kekurangan dan pekerjaan lanjutan:** Uji RESET/RENEW pada Keihin dan
  Shindengen fisik; interpretasi separuh payload nol Keihin; RENEW bukan
  pemasangan kunci baru dan harus dijelaskan pada pengguna.

---

## 5. Tuning dan Remap dengan FTDI

- **Tujuan:** Menyediakan alur Tuning/Remap dengan operasi nyata, konfirmasi
  bertipe, dan label bukti yang jujur.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/core/remap/*`
  - `app/src/main/java/com/revcore/android/data/maps/RemapExecutionGuard.kt`
  - `app/src/main/java/com/revcore/android/ui/RemapOperationState.kt`
  - `app/src/main/java/com/revcore/android/ui/screens/TuningScreen.kt`
- **Implementasi aktual:**
  - Read/write/verify dengan urutan frame rekonstruksi, typed `TULIS`,
    typed confirmation untuk reset ECM.
  - Write yang dimulai tidak auto-restore setelah gagal; `recoveryRequired`
    ditampilkan.
  - UI tiga bagian: Kendaraan & ECM → MAP → BIN & proses remap.
  - MAP selection/export tetap ada; Write BIN utama dinonaktifkan selama
    `remapSupported` false dan menampilkan "DUKUNGAN REMAP BELUM TERSEDIA".
  - `RemapWorkflow.WRITE_HARDWARE_VALIDATED = false`.
- **Hasil test:** `RemapEngineTest`, `RemapCodecTest`, `RemapChecksumTest`,
  `RemapSeedKeyTest`, `RemapVendorStrategiesTest`, `RemapExecutionGuardTest`,
  `RemapWorkflowTest`; suite penuh lulus.
- **Bukti penelitian:** Frame Keihin/Shindengen dari dekompilasi; referensi
  `refs/wrt/`; laporan `REMAP_UI_INVENTORY.md`.
- **Status implementasi:** TERIMPLEMENTASI SEBAGIAN — operasi host tersedia,
  protokol tulis belum tervalidasi.
- **Status validasi kendaraan:** BELUM DIUJI; tidak ada write ke ECU fisik.
- **Kekurangan dan pekerjaan lanjutan:** Validasi ECU, offset checksum per
  profil, fast 62.500 baud, geometri halaman, recovery power-loss, dan bukti
  read-back penuh. Jangan mengklaim Write BIN berhasil.

---

## 6. Protocol Manager dan kestabilan USB

- **Tujuan:** Satu pemilik jalur kawat, status per protokol, dan deteksi koneksi
  yang sudah mati.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/core/protocol/ProtocolManager.kt`
  - `app/src/main/java/com/revcore/android/core/protocol/HondaProtocolEngine.kt`
  - `app/src/main/java/com/revcore/android/core/usb/*`
  - `app/src/main/java/com/revcore/android/core/ftdi/FtdiSerialDriver.kt`
- **Implementasi aktual:**
  - `ManagedProtocol`: USB_LINK, ECU_IDENTIFICATION, LIVE_DATA, DTC, KEYLESS_ID,
    REMAP.
  - Lease eksklusif untuk Keyless dan Remap; Live Data tidak boleh ACTIVE saat
    lease dipegang.
  - `requireLiveTransport` memeriksa driver, device, dan `deviceList` sebelum
    setiap transaksi.
  - Tiga kegagalan I/O berturut-turut menandai transport tertutup.
  - Reset ECU membatalkan sesi ECU lama tetapi adapter tetap terbuka.
- **Hasil test:** `ProtocolManagerTest` 7 test; `AdapterConnectionPolicyTest`,
  `KlineTransceiverTest`, `BaselineProtocolTest`; suite penuh lulus.
- **Bukti penelitian:** Audit FTDI/K-Line 2026-11-20; laporan
  `docs/communication-core-replacement.md`; analisis `libecuwb_ui.so`.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI pada adapter/kendaraan fisik.
- **Kekurangan dan pekerjaan lanjutan:** Uji cabut-pasang USB, permission,
  timeout, dan recovery pada perangkat; perilaku ROM tertentu.

---

## 7. Layar tetap menyala ketika RevCore aktif

- **Tujuan:** Layar tidak tidur selama aplikasi terlihat, dan flag dilepas saat
  aplikasi tidak terlihat.
- **Source terkait:** `app/src/main/java/com/revcore/android/MainActivity.kt`.
- **Implementasi aktual:** `FLAG_KEEP_SCREEN_ON` ditambahkan di `onResume` dan
  dihapus di `onPause`; berlaku untuk semua menu, bukan satu layar.
- **Hasil test:** Kompilasi; tidak ada unit test UI/lifecycle.
- **Bukti penelitian:** Laporan Task 7 `CHANGE_REPORT.md`.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI pada perangkat.
- **Kekurangan dan pekerjaan lanjutan:** Konfirmasi perilaku pada Poco X3 NFC
  (MIUI dapat mengubah kebijakan layar).

---

## 8. Gate 10, jeda 2 detik, animasi loading, lalu menu utama

- **Tujuan:** Setiap startup menampilkan jeda 2 detik dan loading, termasuk
  jalur trust record dan jalur gagal; Gate 10 tetap utuh.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`
  - `app/src/main/java/com/revcore/android/MainActivity.kt`
  - `app/src/main/java/com/revcore/android/security/*`
- **Implementasi aktual:**
  - `StartupTiming.HOLD_BEFORE_LOADING_MS = 2000`.
  - `failAfterHold`: Preparing → hold 2 detik → Starting(1f) → 500 ms →
    Failed.
  - `acceptTrustedReport` memakai `awaitHoldAndInitialize`: trust record
    melewati exam/dialog, tetapi tidak melewati hold.
  - `runInitialization` langsung memasang `Starting(0f)` setelah hold.
  - Debug build yang lulus penuh tetap melewati konfirmasi dan hold; tidak
    menulis trust record rilis.
- **Hasil test:** `StartupTimingTest` 3; `IntegrityGateTest` 31;
  `GateStagingTest` 13; `TrustRecordTest` 11; `InstallationMarkerTest` 10;
  suite penuh lulus.
- **Bukti penelitian:** Audit startup/UI integrity 2026-11-14–16; laporan
  `REVCORE_STARTUP_PERFORMANCE_REPORT.md`, `REVCORE_TRUST_RECORD_LIFECYCLE_REPORT.md`.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI; runtime Keystore/trust record pada
  perangkat belum diverifikasi.
- **Kekurangan dan pekerjaan lanjutan:** Uji startup pertama, startup dengan
  trust record, dan startup gagal pada perangkat; jangan mengubah urutan yang
  sudah disetujui.

---

## 9. Susunan UI DTC

- **Tujuan:** Scan di atas, daftar DTC di tengah, Hapus DTC di bawah.
- **Source terkait:** `app/src/main/java/com/revcore/android/ui/screens/DiagnosticsScreen.kt`.
- **Implementasi aktual:**
  - Kartu Scan di bagian atas.
  - Daftar DTC dalam `LazyColumn` di tengah.
  - Tombol "HAPUS DTC" di bawah, tetap dapat dijangkau setelah scroll.
  - Clear hanya `SUCCESS_VERIFIED` bila ECU mengonfirmasi.
- **Hasil test:** Kompilasi; tidak ada unit test layout.
- **Bukti penelitian:** Laporan Task 9; inspeksi layar.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** BELUM DIUJI; tampilan dan perilaku DTC pada
  kendaraan belum dikonfirmasi.
- **Kekurangan dan pekerjaan lanjutan:** Uji layout pada layar kecil, perilaku
  scroll, dan status MIL fisik.

---

## 10. Audit dan pembersihan identitas internal menjadi RevCore

- **Tujuan:** Menghapus identitas internal pihak lain dari source, tanpa merusak
  identifier teknis eksternal yang diperlukan.
- **Source terkait:**
  - `app/src/main/java/com/revcore/android/data/maps/*`
  - `app/src/main/assets/revcore_profiles.json`
  - `app/src/main/assets/revcore_maps/**`
  - `app/proguard-rules.pro`
  - File test terkait.
- **Implementasi aktual:**
  - Paket `data.flashpro` → `data.maps`; `FlashProProfile` → `RevCoreProfile`;
    `FlashProProfiles` → `RevCoreProfiles`; `FlashProViewType` →
    `RevCoreViewType`; `FlashProDefinitionBuilder` → `RevCoreDefinitionBuilder`.
  - ID profil `flashpro.dat*` → `revcore.dat*`.
  - Aset `flashpro_profiles.json`/`flashpro_maps` → `revcore_profiles.json`/
    `revcore_maps` (185 rename).
  - Evolusi keputusan: `IDENTITY_AUDIT.md` (Task 10 awal) masih mempertahankan
    nama aset `flashpro_*`; order ABSOLUTE No.3 pada hari yang sama kemudian
    memutuskan rename aset dan ID profil, lalu regenerasi policy Gate 10.
    Status yang berlaku adalah yang terakhir: `revcore_*`.
  - Identifier eksternal yang tetap dipertahankan adalah istilah teknis dan
    attribution yang diperlukan: `FTDI`, `FT232R`, `K-Line`, `Keihin`,
    `Shindengen`, `Honda`, `VomConfig`/`BinLoader` (KDoc provenance), serta nama
    berkas riset di luar aplikasi.
- **Hasil test:** Kompilasi, unit test penuh, `assembleDebug`; DEX current
  memuat `com/revcore/android/data/maps`; policy Gate 10 165 path = 165 aset;
  0 nama `flashpro` di payload policy.
- **Bukti penelitian:** `IDENTITY_AUDIT.md`; laporan ABSOLUTE bagian IV.
- **Status implementasi:** TERIMPLEMENTASI (software).
- **Status validasi kendaraan:** Tidak relevan untuk rename; validasi perangkat
  tetap pending untuk fitur.
- **Kekurangan dan pekerjaan lanjutan:** Pastikan dokumen/laporan lama yang
  masih memakai nama lama tidak dianggap source; jangan menghapus identifier
  eksternal yang diperlukan protokol.

---

## Ringkasan status

| No. | Pekerjaan | Implementasi | Test software | Validasi kendaraan |
|---|---|---|---|---|
| 1 | Unified Live Data | Selesai | Lulus | Pending |
| 2 | Prakiraan AFR | Selesai | Lulus | Pending |
| 3 | Log TXT & event | Selesai | Sebagian | Pending |
| 4 | Keyless ID | Selesai | Lulus | Pending |
| 5 | Tuning/Remap | Sebagian | Lulus | Pending |
| 6 | Protocol Manager | Selesai | Lulus | Pending |
| 7 | Layar tetap menyala | Selesai | Tidak ada unit test | Pending |
| 8 | Startup Gate 10 | Selesai | Lulus | Pending |
| 9 | UI DTC | Selesai | Kompilasi saja | Pending |
| 10 | Identitas RevCore | Selesai | Lulus | Tidak relevan |

**Keputusan penerimaan akhir tetap milik Otoy.**
