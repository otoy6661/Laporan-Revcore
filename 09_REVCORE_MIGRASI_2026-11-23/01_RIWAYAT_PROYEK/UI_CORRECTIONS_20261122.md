# EMPAT KOREKSI UI TERBARU — KEPUTUSAN OTOY

**Tanggal keputusan:** 2026-11-22
**Status:** aturan aktif
**Sumber teks:** perintah Owner "MASTER WORK ORDER — REVCORE REBUILD 161126 —
ABSOLUTE CORRECTION & IMPLEMENTATION ORDER" dan "WORK ORDER — KOREKSI RELEASE
REVCORE"
**Pelaksana:** Kodir melalui Qoder

Dokumen ini mencatat keputusan apa adanya, lalu status implementasi yang dapat
dibuktikan dari source dan laporan. Teks keputusan tidak boleh ditulis ulang
menjadi klaim yang lebih kuat daripada buktinya.

---

## No. 1 — Startup

### Keputusan Owner (verbatim)

> SUDAH SESUAI. WAJIB DIPERTAHANKAN.
>
> Gate 10 atau trust record valid → jeda 2 detik → animasi loading RevCore → menu utama.
>
> Loading harus selalu muncul setiap startup. Trust record valid boleh melewati konfirmasi Gate 10, tetapi tidak boleh melewati loading.
>
> Jangan mengubah desain atau urutan yang telah disetujui.

### Status implementasi

**SELESAI di kode + unit test; belum diuji pada perangkat.**

Bukti source:

- `StartupTiming.HOLD_BEFORE_LOADING_MS = 2000` di
  `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`.
- `failAfterHold`: Preparing → 2000 ms → Starting(1f) → 500 ms → Failed.
- `acceptTrustedReport` memakai `awaitHoldAndInitialize`: trust record melewati
  exam/dialog, tetapi tidak melewati hold.
- `runInitialization` memasang `Starting(0f)` segera setelah hold.
- `MainActivity` memetakan `Idle`/`Preparing` ke `StartupHoldScreen` dan
  `Starting` ke `LoadingScreen`.

Bukti test: `StartupTimingTest` 3 test; suite Gate/startup lulus.

Batas: runtime Keystore/trust record dan tampilan nyata pada perangkat belum
diverifikasi. Jangan mengubah desain atau urutan ini tanpa perintah Owner.

---

## No. 2 — Live Data

### Keputusan Owner (verbatim)

> - Sebelum kendaraan terhubung, jangan tampilkan tabel kosong dan pesan debugging.
> - Setelah kendaraan terhubung, tampilkan parameter yang berhasil dibaca dan lolos validasi.
> - Setelah mesin hidup, parameter tambahan muncul otomatis.
> - Pertahankan parameter sebelumnya.
> - Prakiraan AFR baru muncul setelah O2 valid diterima.
> - Jangan mengganti sistem pembacaan dan polling yang sudah ada.
> - Jangan mengarang angka atau parameter.

### Status implementasi

**SELESAI di kode + unit test; belum diuji pada kendaraan.**

Bukti source:

- `LiveDataPresentation.pane`: `CONNECT_VEHICLE` → `WAITING_FOR_DATA` → `LIVE`.
- `UnifiedLiveTable.build()` hanya menampilkan sensor yang benar-benar ada di
  `parameterStates`/`readings`.
- `LiveDataScreen` menampilkan kartu instruksi/menunggu, bukan tabel kosong.
- Retensi `STALE` di `LiveParameterStates.retainPreviousReadings`; nilai lama
  mempertahankan timestamp asli dan tidak berubah menjadi VALID.
- `LiveDataPresentation.showsAfr` hanya `O2 == VALID`; AFR tidak ditampilkan
  saat O2 STALE.
- Polling lama (`HondaProtocolEngine`, `HondaLiveDataProfiles`) tidak diganti.

Bukti test: `LiveDataPresentationTest` 5, `UnifiedLiveTableTest` 8,
`LiveDataRetentionTest` 6, `AfrEstimatorTest` 7.

Batas: respons ECU nyata, transisi mesin mati/hidup, dan nilai O2 aktual belum
diuji. Jangan mengganti mekanisme polling yang sudah ada.

---

## No. 3 — Keyless ID

### Keputusan Owner (verbatim)

> Susunan UI:
>
> 1. Kendaraan Teridentifikasi.
> 2. ECM: KEIHIN atau SHINDENGEN.
> 3. BACA KEY ID.
> 4. RESET KEY ID.
> 5. RENEW KEY ID.
>
> Baca ID harus berhasil sebelum tombol Reset dan Renew aktif.
>
> Popup ID harus menyediakan fungsi salin.
>
> Protokol, urutan perintah, dan prosedur kontak ON/OFF harus mengikuti source dan penelitian masing-masing ECM.
>
> ACK saja bukan bukti keberhasilan.

### Status implementasi

**SELESAI di kode + unit test; belum diuji pada kendaraan.**

Bukti source:

- `KeylessScreen.kt` menyusun lima bagian bernomor sesuai urutan.
- `KeylessUiState.canRunMaintenance` mensyaratkan adapter, tidak sibuk, profil
  cocok, `readResultValid`, dan report baca sukses.
- Popup ID menyediakan tombol salin.
- Read/Reset/Renew memakai engine `core/keyless/` dengan frame hasil firmware.
- `KeylessResetEngine` membaca ulang target; ACK saja tidak menghasilkan
  `SUCCESS`.

Bukti test: `KeylessUiStateTest` 7, `KeylessFrameCodecTest` 13,
`KeylessIdDecoderTest` 7, `KeylessReadEngineTest` 6,
`KeylessResetCodecTest` 8, `KeylessResetEngineTest` 7,
`KeylessRenewPlanTest` 4.

Batas dan drift:

- RESET/RENEW pada ECU fisik belum diuji.
- `RENEW_SUPPORTED = true` di source current; dokumen delivery lama masih
  menyebut `false`. Ini dicatat di `10_KNOWN_ISSUES/UNRESOLVED_AND_LIMITS.md`.
- RENEW adalah urutan zero-write yang sama dengan RESET, bukan penulisan material
  kunci baru. UI harus tetap menjelaskan hal ini.

---

## No. 4 — Tuning

### Keputusan Owner (verbatim)

> UI Tuning harus mengikuti pola tata letak dan alur FlashPro sebagai referensi teknis, tetapi tetap memiliki identitas visual RevCore.
>
> Jangan mengubah tema utama RevCore secara sepihak.
>
> Jangan sekadar menempelkan fitur tanpa integrasi UI.
>
> Jangan menghapus pemilihan dan ekspor MAP.
>
> Jangan mengklaim Write BIN berhasil jika protokol belum terverifikasi.

### Status implementasi

**SELESAI di kode + unit test; validasi protokol/ECU belum ada.**

Bukti source:

- `TuningScreen.kt` menyusun tiga bagian bernomor:
  1. Kendaraan & ECM
  2. MAP
  3. BIN & proses remap
- MAP dipilih sebelum operasi BIN; pemilihan dan ekspor MAP tetap ada
  (`btn_export_map`).
- Write BIN pada alur MAP memakai `enabled = remapSupported`; bila belum
  didukung, tampil "DUKUNGAN REMAP BELUM TERSEDIA — Write BIN tidak aktif."
- Alur remap memiliki Read/Verify/Write dengan konfirmasi bertipe dan status
  bukti; write tidak dianggap sukses hanya karena ACK.
- Advanced/ROM tools ditampilkan dalam satu bagian yang dapat dibuka.

Bukti test: seluruh suite Tuning/Map/Remap lulus; testTag lama dipertahankan.

Batas:

- `RemapWorkflow.WRITE_HARDWARE_VALIDATED = false`.
- Protokol tulis belum diuji pada ECU fisik; offsets checksum per profil, fast
  baud, geometri halaman, dan recovery power-loss belum terbukti.
- Jangan mengklaim Write BIN berhasil dan jangan menghapus ekspor MAP.

---

## Ringkasan kepatuhan

| Koreksi | Status | Bukti | Yang belum |
|---|---|---|---|
| 1 Startup | Sesuai, dipertahankan | `StartupTimingTest`, `MainViewModel` | Uji perangkat |
| 2 Live Data | Terimplementasi | `LiveDataPresentation`, `UnifiedLiveTable`, retensi | Uji kendaraan |
| 3 Keyless | Terimplementasi | `KeylessScreen`, `KeylessUiState`, engine | Uji ECU/smartkey |
| 4 Tuning | Terimplementasi | `TuningScreen`, guard/remap | Uji protokol/ECU |

**Penerimaan akhir tetap milik Otoy.**
