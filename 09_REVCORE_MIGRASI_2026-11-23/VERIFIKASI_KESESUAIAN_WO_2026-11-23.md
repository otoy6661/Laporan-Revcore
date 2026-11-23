# Verifikasi Kesesuaian Work Order — RevCore (2026-11-23)

Sifat: verifikasi **baca-saja** terhadap source current. Tidak ada perubahan source, build, signing, commit, atau pengujian perangkat dalam pekerjaan verifikasi ini. Keputusan atas temuan tetap milik Otoy.

Work order acuan: MASTER WORK ORDER — REVCORE (FULL SOURCE AUDIT, UI CORRECTION, PROTOCOL IMPLEMENTATION & OFFICIAL SIGNED RELEASE APK), pasal A–J.

Source yang diperiksa: working tree `RevCore_work_live_keyless_20261120`, branch `rebuild-161126` (HEAD `1cc9120`), keadaan belum-commit (23 untracked, 25 modified, 164 rename, 21 rename+modified). APK 161126 yang telah ada tidak diubah dan tidak ditandatangani ulang.

## Ringkasan

| Pasal | Status | Keterangan singkat |
|---|---|---|
| A. Mandat | Dipatuhi | Verifikasi baca-saja; tanpa perubahan dan tanpa build |
| C1. H-Diag Live Data | **GAP bukti** | Matriks evidence menandai pemetaan H-Diag `UNSUPPORTED` |
| C2. FlashPro Keyless | Sesuai kode, dokumen kedaluwarsa | Read/Reset/Renew ada; matriks masih menulis RENEW `UNSUPPORTED` |
| C3. Remap | Sesuai + batas jelas | Write diimplementasikan tetapi belum divalidasi hardware |
| C4. Matriks pemetaan | Ada, sebagian kedaluwarsa | Baris 15 dan 28 perlu dikoreksi |
| D1. Card AFR | **GAP** | Format bukan `XX.XX : 1`; masih ada teks rumus teknis |
| D2. Card Live Data dinamis | Sesuai | RPM teratas; hanya parameter valid; retensi ada |
| D3. Keyless UI | **GAP** | Label bukti internal tampil di UI normal |
| D4. Tuning/Remap UI | Sesuai keputusan terbaru | Write BIN terkunci; lihat catatan konflik E3 |
| D5. DTC UI | Sesuai | Scan atas, daftar scroll, Hapus bawah |
| D6. Konsistensi global | Sesuai dengan catatan | Tema konsisten; teks internal lain masih ada (lihat D1/D3) |
| D7. Startup/loading | Terimplementasi, DEVICE_UNTESTED | Jeda 2 detik lalu loading pada semua jalur |
| E1. Live Data | Terimplementasi, DEVICE_UNTESTED | Scheduler, pencocokan response, retensi, decoder per tabel |
| E2. Keyless ID | Terimplementasi, DEVICE_UNTESTED | Reset/Renew aktif dengan prasyarat; readback sesuai prosedur |
| E3. Remap | Konflik dengan D4 — keputusan 2026-11-22 berlaku | Write terkunci sampai validasi |
| E4. Protocol Manager/FTDI | Terimplementasi, DEVICE_UNTESTED | Lease tunggal, timeout, foreground/background |
| F. Gate 10/identitas | Tidak diubah | Detail internal tidak dipublikasikan |
| G. Pengujian | Software lulus; perangkat belum | 55 suite, 457 tes, 0 gagal/error |
| H. APK Release resmi | Sudah ada (161126/221126) | Tidak dibangun/ditandatangani ulang dalam verifikasi ini |
| I. Laporan akhir | Dokumen tersedia | Lihat folder `03_TES_DAN_BUILD/` |
| J. Batas wewenang | Dipatuhi | Tanpa push ke repo aplikasi, tanpa publikasi APK |

## A. Mandat

Work order meminta audit, koreksi UI, implementasi, pengujian, dan build resmi. Verifikasi ini hanya memeriksa kesesuaian; tidak ada source yang diubah dan tidak ada APK baru.

## C. Audit source protokol

### C1. H-Diag — Live Data
`delivery/protocol_evidence_matrix.md` baris 28 menyatakan pemetaan parameter/actuator H-Diag `UNSUPPORTED` (tidak ada mapping yang dapat ditelusuri). Sementara itu decoder Live Data RevCore memakai tabel/profil yang ada, bukan pemetaan H-Diag. Status: **GAP bukti** — work order meminta mapping yang dapat ditelusuri; sampai matriks diperbarui dengan bukti, klaim kesesuaian H-Diag tidak dapat dibuat.

### C2. FlashPro — Keyless ID
- `app/src/main/java/com/revcore/android/core/keyless/KeylessRenewPlan.kt`: `RESET_SUPPORTED = true`, `RENEW_SUPPORTED = true`, `RESET_EVIDENCE = RECONSTRUCTED_FROM_BINARY`.
- `app/src/main/java/com/revcore/android/ui/keyless/KeylessUiState.kt:100`: `canRunMaintenance` mensyaratkan adapter terhubung, identitas cocok, hasil baca valid, dan operasi sukses.
- Urutan command, panjang data, dan readback mengikuti prosedur yang direkonstruksi; tidak ada klaim sukses hanya dari ACK.
- **Dokumen kedaluwarsa:** `delivery/protocol_evidence_matrix.md` baris 15 masih menulis RENEW `UNSUPPORTED` dan `RENEW_SUPPORTED=false`, berlawanan dengan source current. Dokumen lama tidak diedit; perbedaan ini dicatat.

### C3. Tuning / Remap
- `app/src/main/java/com/revcore/android/data/maps/RemapWorkflow.kt`: `WRITE_IMPLEMENTED = true`, `WRITE_HARDWARE_VALIDATED = false`, `readyForWrite = WRITE_IMPLEMENTED && WRITE_HARDWARE_VALIDATED` (selalu false saat ini).
- Transfer, pemeriksaan response, dan penanganan kegagalan ada; klaim "Write BIN berhasil" tidak dibuat (lihat teks "Write BIN tidak aktif" di `TuningScreen.kt:792` dan `:1242`).

### C4. Pemetaan source ke RevCore
Matriks ada dan memuat status bukti per item. Baris 15 (RENEW) kedaluwarsa; baris 28 (H-Diag) tetap `UNSUPPORTED`; baris 6 (AFR) berstatus `HYPOTHESIS` yang memang sesuai sifat rumus. Status: **perlu koreksi dokumen**, bukan koreksi source.

## D. Perbaikan UI

### D1. Card Prakiraan AFR — GAP
`app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt:690-716` menampilkan nilai `String.format("%.2f", estimate.afr)` dengan label "AFR", lalu teks teknis "Input: tegangan O2 … Rumus: AFR = 17.700001 − 5.351 × V_O2".

Work order meminta format `XX.XX : 1`, gaya card ringkas, dan tanpa teks teknis/debugging. Card memang hanya muncul saat O2 valid (`showsAfr` di `ui/livedata/LiveDataPresentation.kt:50`), jadi gating sudah benar; yang belum sesuai adalah format dan teks teknis.

### D2. Susunan Live Data Card — SESUAI
- Urutan tampilan dimulai RPM (`LiveDataScreen.kt:890`), RPM selalu di atas.
- Hanya parameter `present` yang dirender (`core/comm/UnifiedLiveTable.kt:42`).
- Retensi card valid ada: `core/comm/LiveDataProfile.kt:75` (`retainPreviousReadings`) dipakai di `core/protocol/HondaProtocolEngine.kt:1531`; data retensi menjadi STALE dan tidak dipakai sebagai AFR terbaru.
- Penyimpangan lama (card literal RPM-only) sudah diperbaiki melalui perintah koreksi 2026-11-22.

### D3. Keyless ID UI — GAP
`ui/screens/KeylessScreen.kt:670-671` menampilkan badge `PROTOCOL_ANALYZED` dan `FIELD_TEST_PENDING`; `:996` menampilkan teks "Status protokol: PROTOCOL_ANALYZED…" di UI normal. Work order melarang menampilkan label internal seperti `FIELD_TEST_PENDING`/`RECONSTRUCTED_FROM_BINARY` kepada pengguna normal. Fungsi Read/Reset/Renew dan popup hasil yang dapat disalin sudah ada.

### D4. Tuning / Remap UI — SESUAI KEPUTUSAN TERBARU
Identitas RevCore dipertahankan, pemilihan MAP dan export tetap ada, status baca/tulis jelas, dan tidak ada klaim Write BIN. Write terkunci — lihat catatan konflik di E3.

### D5. DTC UI — SESUAI
`ui/screens/DiagnosticsScreen.kt`: tombol Scan di atas (`:133`), daftar DTC dalam `LazyColumn` (`:262`), tombol Hapus di bawah (`:275`).

### D6. Konsistensi UI Global — SESUAI DENGAN CATATAN
Tema dan warna RevCore dipertahankan; card mengikuti gaya yang ada. Namun aturan "jangan menampilkan status internal" belum terpenuhi di D1 dan D3; perbaikan keduanya akan menutup D6.

### D7. Startup dan Loading — TERIMPLEMENTASI
`ui/MainViewModel.kt`: `HOLD_BEFORE_LOADING_MS = 2000L`; `awaitHoldAndInitialize` menahan 2 detik setelah laporan/konfirmasi lalu memulai `Starting(0f)`; `runInitialization` dijalankan setelahnya. Jalur trust record dan jalur gagal memakai tahanan yang sama (`failAfterHold`). Unit test startup mencakup urutan ini. Tampilan nyata dan runtime perangkat: **DEVICE_UNTESTED**.

## E. Implementasi sistem dan protokol

- **E1 Live Data:** `core/comm/LivePollPolicy.kt` mengatur pergantian request; pencocokan response, penanganan panjang berbeda, dan decoder per tabel ada di `core/comm/`; retensi dan pemisahan STALE ada. Pengujian fisik ECU: belum.
- **E2 Keyless ID:** prosedur Keihin/Shindengen dipisah di `core/keyless/`; prasyarat operasi divalidasi; hasil baca divalidasi; readback sesuai prosedur. Pengujian kendaraan: belum.
- **E3 Remap:** alur MAP read/select/export dan persiapan transfer ada. **Konflik tertulis:** E3 melarang mengunci fungsi hanya karena belum diuji fisik, sedangkan perintah koreksi 2026-11-22 (dan laporan rebuild) memutuskan write tetap terkunci sampai `isVerifiedRemap`. Sebagai perintah yang lebih baru, keputusan 2026-11-22 berlaku. Work order ini tidak membatalkannya secara eksplisit.
- **E4 Protocol Manager/FTDI:** `core/protocol/ProtocolManager.kt` memegang lease tunggal (`acquireExclusive` `:237`, `releaseExclusive` `:273`), timeout, deteksi disconnect, dan pemulihan; tidak ada guard tambahan tanpa alasan teknis yang ditemukan.

## F. Gate 10 dan identitas internal

Kebijakan, inventaris aset, pin sertifikat, dan alur startup tidak diubah oleh verifikasi ini. Sesuai keputusan pemilik, interno Gate 10 tidak dipublikasikan di repo ini dan tidak dijelaskan dalam laporan ini.

## G. Pengujian

- Build/test terakhir yang tercatat: 55 suite, 457 tes, 0 gagal/error (lihat `03_TES_DAN_BUILD/test-results-current/` dan laporan tes).
- Bukti APK: 161126 `b2b6c5bd…7432` (39.689.642 B), 221126 `cb5892a3…ba556` (39.841.086 B); v2+v3, tidak debuggable, cert pin `c649cb4f…d098`.
- Pengujian visual UI dan perangkat fisik (Poco X3 NFC, USB-OTG/K-Line, ECU) **belum dilakukan** dalam verifikasi ini. Klaim penerimaan hanya boleh muncul setelah laporan pengujian dari Otoy.

## H. APK Release resmi

APK Release resmi sudah ada dari pekerjaan sebelumnya dan tidak dibangun ulang dalam verifikasi ini. Tidak ada signing key baru, tidak ada perubahan versionName/versionCode/applicationId, dan tidak ada kata sandi yang diminta, dibaca, atau dicatat.

## I. Laporan akhir

Laporan audit, rebuild, UI, tes, signing, dan verifikasi paket tersedia di folder ini. Dokumen berstatus titik-waktu dan tidak menggantikan pengujian perangkat.

## Temuan yang memerlukan keputusan Otoy

1. **D1** — ubah format card AFR menjadi `XX.XX : 1` dan hapus teks rumus teknis dari UI normal.
2. **D3** — sembunyikan label bukti internal (`PROTOCOL_ANALYZED`, `FIELD_TEST_PENDING`, dan sejenisnya) dari UI normal.
3. **C1** — lengkapi atau turunkan klaim pemetaan H-Diag di matriks evidence.
4. **C4** — perbarui baris 15 matriks (RENEW) agar sesuai source, atau tandai dokumen sebagai kedaluwarsa secara eksplisit.
5. **E3 vs D4** — konfirmasi bahwa write lock tetap berlaku sampai validasi perangkat.

Tidak ada perubahan yang dilakukan atas lima temuan tersebut. Menunggu instruksi Otoy.
