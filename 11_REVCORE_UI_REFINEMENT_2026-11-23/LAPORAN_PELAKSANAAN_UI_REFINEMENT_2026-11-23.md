# RevCore — Pelaksanaan Work Order UI Refinement & Official Signed Release (2026-11-23)

Laporan pelaksanaan. Titik awal: Build 231126 (SHA-256 `2b4122aa…`). Hasil: build resmi baru
`RevCore build 231126.apk` (SHA-256 `eb94f159…`); build sebelumnya diarsipkan di
`dist/archive/RevCore build 231126_162833.apk` dan tidak dihapus.

## 1. Ringkasan perubahan UI

| Butir | Perubahan yang dilakukan |
|---|---|
| A. Startup | Urutan tetap Gate 10 → dialog KONFIRMASI → jeda 2 detik → Loading → menu. Loading kini punya jendela tampil sendiri (min. 2.600 ms, hold selesai 400 ms, floor jalur gagal 1.200 ms), animasi dihaluskan (tween 700 ms FastOutSlowIn, glow 2.400 ms). Logika Gate 10 tidak diubah. |
| B. Keyless ID | Nama "Keyless ID"; urutan 1 KENDARAAN TERIDENTIFIKASI → 2 ECM → 3 BACA KEY ID → 4 RESET KEY ID → 5 RENEW KEY ID. Semua label "BATASAN BUKTI", "Detail teknis", dan kartu batasan dihapus dari layar. RESET/RENEW hanya aktif setelah BACA KEY ID sukses dan hasil masih valid; verifikasi tetap lewat read-back. Identitas kendaraan hanya dari pencocokan database, tanpa pemilihan model manual. |
| C. Tuning | Alur enam tahap: ECU CONNECTION → MAP FILE → MAP EDITOR → SAVE MAP → ECU PROGRAMMING → OPERATION STATUS. Editor mempertahankan tabel, grafik 2D (ListLineChart), grafik 3D (baru), undo/redo, CMP before/after. Kartu SIMPAN MAP (EKSPOR MAP / VERIFIKASI CHECKSUM) dan kartu OPERATION STATUS dipisah. Tidak ada gate tulis baru; Gate 10 tidak masuk alur tulis ECU. |
| D. Live Data | Kartu RPM selalu paling atas dan satu-satunya sebelum koneksi; setelah koneksi + kontak ON hanya parameter yang benar-benar terbaca valid yang tampil; data basi ditandai STALE. Sudah sesuai sebelum work order; diverifikasi ulang dari source. |
| E. AFR | Kartu ringkas format `XX.XX : 1`, hanya saat hasil estimasi O2 valid; teks kaveat teknis dihapus. Rumus estimasi tidak diubah. |
| F. DTC | Tombol Scan di atas, daftar scroll di tengah, Delete di bawah. Sudah sesuai; diverifikasi ulang dari source. |
| G. Konsistensi | Identitas, logo, tema, dan branding tidak diubah; tidak ada fitur dihapus (alat ROM lama tetap di bagian Advanced). |

## 2. Source yang benar-benar diubah

| Berkas | Perubahan |
|---|---|
| `app/src/main/java/com/revcore/android/ui/MainViewModel.kt` | Konstanta `StartupTiming` (jeda 2.000 ms, jendela loading 2.600/400/1.200 ms) dan penerapan jendela loading di `runInitialization()` via `awaitLoadingWindow()`. |
| `app/src/main/java/com/revcore/android/ui/screens/LoadingScreen.kt` | Animasi progress/glow diperlambat dan dihaluskan (tween 700 ms eased; glow 2.400 ms). |
| `app/src/main/java/com/revcore/android/ui/screens/IntegritySuccessDialog.kt` | Teks dialog menjelaskan urutan: KONFIRMASI → Loading beberapa saat → halaman utama. |
| `app/src/main/java/com/revcore/android/ui/screens/KeylessScreen.kt` | Penghapusan kartu "BATASAN BUKTI"/detail teknis, penyederhanaan laporan pemeliharaan, tombol RESET/RENEW + teks gating, teks dialog RENEW, pembersihan 8 import tak terpakai. |
| `app/src/main/java/com/revcore/android/ui/screens/TuningScreen.kt` | Restrukturisasi enam seksi, `MapSaveCard`, `OperationStatusCard`, tampilan TABEL/GRAFIK 3D (`Table3DView` baru), pemindahan laporan baca/tulis/verifikasi ke seksi OPERATION STATUS. |
| `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt` | `AfrEstimateCard` ringkas (`%.2f : 1`), kaveat teknis dihapus, import `Locale` ditambahkan. |
| `app/src/test/java/com/revcore/android/ui/StartupTimingTest.kt` | Tes baru: jendela loading harus lebih lama dari jeda 2 detik dan terpisah darinya. |

Tidak ada perubahan: `applicationId`, `versionName` 1.0, `versionCode` 1, signing identity,
kebijakan integritas Gate 10, maupun file di luar daftar di atas.

## 3. Hasil test dan build

- `:app:testDebugUnitTest` — BUILD SUCCESSFUL; 461 test, 55 suite, 0 gagal, 0 error
  (Build 231126 sebelumnya 460; tambahan 1 tes StartupTiming).
- Release build resmi via `tools/revcore-official-release-gui.ps1` → status `SUCCESS`
  `2026-11-23T16:28:38+07:00`; password dimasukkan sendiri oleh pemilik melalui GUI,
  tidak dibaca/dicatat.
- Verifikasi APK independen:
  - package `com.revcore.android`, versionCode `1`, versionName `1.0`, targetSdk 36;
  - apksigner: Verifies; v2 `true`, v3 `true`; 1 signer;
  - sertifikat SHA-256 `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`
    (sesuai sertifikat resmi RevCore), DN `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta, ST=DKI Jakarta, C=ID`;
  - atribut `debuggable` tidak ada (bukan debug build);
  - UI/DEX manifest tertanam dan terverifikasi: 20 entri UI, 1 entri DEX, 7 struktur XML;
  - tidak ditemukan material kunci di dalam APK.

## 4. Gate 10

- Urutan startup Gate 10 → konfirmasi → 2 detik → loading → menu dipertahankan; tidak ada
  bypass atau aturan integritas baru.
- `assets/integrity_policy.json` (37.410 byte) dan `assets/ui_integrity_manifest.json`
  (13.338 byte) ada di dalam APK hasil build.
- Gate 10 tetap app-integrity; tidak dijadikan gate tulis ECU.

## 5. Status verifikasi perangkat fisik

**BELUM diverifikasi di perangkat.** Urutan startup, durasi 2 detik, kehalusan animasi
loading, tampilan Keyless, Tuning (termasuk grafik 3D), Live Data, AFR, dan DTC baru
diverifikasi lewat kompilasi + unit test + pembacaan source. Pemilik memverifikasi sendiri
pada Poco X3 NFC (Android 12/MIUI 14). Tidak ada klaim penerimaan UI.

## 6. Kendala / belum terverifikasi

1. Lampiran video `video6061917185171792154.mp4` (acuan animasi loading) tidak dapat dibuka
   oleh executor: tidak ada ffmpeg dan browser menolak `file://`. Animasi disesuaikan dari
   deskripsi tekstual work order, bukan bingkai video. Kesesuaian visual menunggu penilaian pemilik.
2. Verifikasi perangkat fisik seluruh butir A–G belum dilakukan (lihat bagian 5).
3. Grafik 3D Tuning (`Table3DView`) diuji kompilasi dan unit test saja; belum dilihat di perangkat.
4. RESET/RENEW Keyless tetap bergantung prosedur kontak/adapter; belum diuji pada ECU nyata
   dalam sesi ini dan tidak ada klaim keberhasilan tulis.
5. Grafik 3D menampilkan nilai sel dari parser MAP yang ada; tidak ada struktur MAP,
   checksum, atau alamat baru yang dikarang.

## 7. APK hasil

- Nama: `RevCore build 231126.apk` (lokal di work tree `dist/`; tidak diunggah ke repo laporan)
- Ukuran: 39.841.086 byte
- SHA-256: `eb94f159fa32484b1923516acd4ef024e8283bd0c686bbd20af1133d623dedc9`
- Signing: resmi RevCore, v2+v3, bukan debug; sertifikat
  `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`
- APK Build 231126 lama (`2b4122aa…`) diarsipkan ke `dist/archive/RevCore build 231126_162833.apk`

Bukti: folder `BUKTI/`.
