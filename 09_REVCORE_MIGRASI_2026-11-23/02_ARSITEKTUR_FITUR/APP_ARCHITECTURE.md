# ARSITEKTUR APLIKASI REVCORE

**Package:** `com.revcore.android`
**Versi:** `1.0` (`versionCode 1`)
**Stack:** Kotlin, Jetpack Compose, minSdk 26, targetSdk/compileSdk 36, JVM 21
**Build:** Gradle 9.6.0 offline, R8 minify + resource shrinking
**Source acuan:** `02_CURRENT_WORKING_STATE/project`

---

## 1. Lapisan

| Lapisan | Paket | Tanggung jawab |
|---|---|---|
| Entry point | `MainActivity.kt` | lifecycle, USB intent, keep-screen, routing startup, render layar |
| Koordinator | `ui/MainViewModel.kt` | startup, sesi adapter, operasi ECU, state UI, event log |
| UI Compose | `ui/`, `ui/screens/`, `ui/keyless/`, `ui/livedata/`, `ui/tuning/` | layar dan state presentasi |
| Protokol | `core/protocol/` | identifikasi, Live Data, DTC, frame K-Line, sesi, `ProtocolManager` |
| Komunikasi | `core/comm/` | profil ECU, codec frame, wire engine, tabel Live Data, AFR |
| Transport | `core/usb/`, `core/ftdi/` | deteksi USB, permission, FTDI, read/write bulk |
| Keyless | `core/keyless/` | read, reset, renew, codec, profil, read-back |
| Remap | `core/remap/` | codec, seed-key, checksum, engine baca/tulis/verifikasi |
| MAP | `data/maps/`, `data/rom/` | katalog, codec, crypto, parser, editor, validator, ROM |
| Database | `data/ecu/` | model ECM, database, identitas kanonik |
| Penyimpanan | `core/storage/`, `data/local/` | folder Revcore, log, terms |
| Keamanan | `security/` | Gate 1–10, policy, UI manifest, trust record, marker |

`LegacyHondaProtocolEngine.kt` disimpan sebagai jalur legacy, bukan jalur utama.

## 2. Alur startup

```text
MainActivity
  -> MainViewModel.startStartup()
  -> Integrity / Gate 1..10 (staged)
       -> PASS semua: AwaitingAcknowledgement (rilis) atau lanjut (trust record)
       -> GAGAL: failAfterHold
  -> Persetujuan Owner (KONFIRMASI) bila diperlukan
  -> Preparing: hold 2 detik (StartupHoldScreen)
  -> Starting: LoadingScreen, progres langkah nyata
  -> Ready: MainAppScaffold (menu utama)
```

Aturan yang dikunci:

- `HOLD_BEFORE_LOADING_MS = 2000`.
- Trust record/in-process reuse melewati exam dan dialog, tetapi tidak melewati
  hold/loading.
- Jalur gagal: Preparing → 2000 ms → Starting(1f) → 500 ms → Failed.
- `runInitialization` memasang `Starting(0f)` tepat setelah hold.
- Debug build yang lulus penuh tetap menampilkan konfirmasi dan hold; tidak
  menulis trust record rilis.

Gate 10 dan trust record dijelaskan di `04_GATE10_SECURITY/GATE10_EXPLAINED.md`.

## 3. Alur data Live Data

```text
USB/FTDI
  -> KlineTransceiver / KlineWireEngine
  -> HondaProtocolEngine.queryLiveData()
  -> HondaLiveDataProfiles (tabel 17/13/11/10 + ekstensi 16/20/D1)
  -> parseLiveFrame()
       -> validasi per-field
       -> LiveParameterStates.withEngineRunning()
       -> retainPreviousReadings() (nilai lama menjadi STALE)
  -> LiveTelemetry
  -> UnifiedLiveTable.build()
  -> LiveDataScreen
```

Prinsip:

- Hanya field yang lolos validasi yang masuk `readings`.
- Field yang tidak tersedia berstatus, bukan angka nol.
- Nilai lama dipertahankan sebagai `STALE` dengan timestamp asli.
- Sensor engine-only tidak dipertahankan saat mesin mati.
- AFR hanya dari O2 `VALID`; selalu berlabel "Prakiraan AFR".
- Polling lama dipertahankan; tidak ada Role Table.

## 4. Alur perintah dan kepemilikan kawat

`ProtocolManager` mencatat status per protokol dan lease eksklusif:

- `USB_LINK`, `ECU_IDENTIFICATION`, `LIVE_DATA`, `DTC`, `KEYLESS_ID`, `REMAP`.
- Hanya `KEYLESS_ID` dan `REMAP` yang boleh memegang lease eksklusif.
- Saat lease dipegang, Live Data tidak boleh ACTIVE.
- Adapter ditutup/hilang → semua lease dicabut dan status diturunkan.
- `HondaProtocolEngine.requireLiveTransport()` memeriksa driver, device, dan
  `deviceList` sebelum transaksi.
- Tiga kegagalan I/O berturut-turut menandai transport tertutup.

## 5. Keamanan dan identitas

- `security/` menghasilkan `integrity_policy.json` dan
  `ui_integrity_manifest.json` saat build.
- Policy dan manifest memakai envelope `SHA256withRSA` dan pin sertifikat resmi
  `c649cb4f…18d098`.
- Gate 10 memeriksa inventaris aset APK; pengecualian tepat tiga berkas.
- Identitas aplikasi: `RevCore`, `com.revcore.android`, kredit `otoy`.
- `versionName`, `versionCode`, `applicationId`, dan signing tidak boleh diubah
  tanpa perintah Owner.

## 6. Build

Konfigurasi:

- `compileSdk = 36`, `targetSdk = 36`, `minSdk = 26`.
- Java/Kotlin target 21; `JAVA_HOME` memakai JBR Android Studio.
- Gradle 9.6.0, offline, `-Prevcore.publishDist=false` untuk build lokal.
- Rilis resmi hanya melalui `tools/revcore-official-release-gui.ps1` dan
  `tools/build-official-release.ps1`.
- R8 keep rules mencakup model keamanan/Gson yang diperlukan.

Build sukses bukan bukti fitur berhasil. Unit test bukan validasi ECU.

## 7. Hal yang belum terbukti

- Runtime Gate 10/Keystore/trust record pada perangkat.
- Semua perilaku ECU fisik (Live Data, DTC, Keyless, Remap).
- Keep-screen pada ROM MIUI.
- Protokol tulis Remap.
- Restore trust record/marker dari backup.

**Validasi perangkat dan penerimaan akhir milik Otoy.**
