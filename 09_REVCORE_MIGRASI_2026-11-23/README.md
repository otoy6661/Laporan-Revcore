# RevCore — Laporan Migrasi, Rebuild, Tes, dan Verifikasi (2026-11-23)

Status: kumpulan laporan dan bukti. Bukan pernyataan bahwa migrasi disetujui, APK tervalidasi di kendaraan, atau pekerjaan RevCore selesai. Keputusan validasi dan penerimaan akhir tetap milik Otoy.

Tanggal unggah: 2026-11-23. Sumber: paket `REVCORE_AI_MIGRATION_BACKUP` (verifikasi 2026-11-23) ditambah laporan verifikasi kesesuaian work order yang dibuat 2026-11-23 (baca-saja).

## 1. Isi folder

| Folder | Isi |
|---|---|
| `00_MULAI_DI_SINI/` | Titik masuk penerus: START HERE, identitas proyek, otoritas dan aturan pemilik |
| `01_RIWAYAT_PROYEK/` | Sejarah proyek (14 topik), 10 pekerjaan rebuild, koreksi UI 2026-11-22, changelog |
| `02_ARSITEKTUR_FITUR/` | Arsitektur aplikasi, komunikasi USB/FTDI/K-Line, database kendaraan, MAP/remap, Keyless ID, Live Data/AFR/DTC/log |
| `03_TES_DAN_BUILD/` | Laporan build/signing/tes, log build, hasil JUnit baseline dan current (XML), checksum APK |
| `04_MASALAH_TERBUKA/` | Item terbuka, drift dokumen, dan batas bukti |
| `05_ARSIP_RESEARCH/` | Indeks arsip riset (isi arsip tidak dipublikasikan) |
| `06_MANIFEST_MIGRASI/` | Laporan verifikasi paket migrasi dan batas/pengecualiannya |
| `07_GIT_HISTORY/` | Penjelasan cara memulihkan riwayat Git dari bundle (bundle-nya sendiri tidak dipublikasikan) |
| `VERIFIKASI_KESESUAIAN_WO_2026-11-23.md` | Verifikasi baca-saja kesesuaian source terhadap MASTER WORK ORDER — REVCORE (status per pasal, gap terbuka) |

## 2. Yang dikecualikan dari publikasi

Sesuai keputusan pemilik ("Kecualikan Gate 10 + key"), yang **tidak** diunggah:

| Item | Alasan |
|---|---|
| `04_GATE10_SECURITY/` (seluruh isi) | Interno Gate 10: penjelasan, policy/manifest, source keamanan, tes, laporan |
| `05_PRIVATE_KEY_INVENTORY/` (seluruh isi) | Inventaris signing dan prosedur kunci |
| `09_TESTS_AND_BUILD/reports/verify_gate10.ps1` | Skrip verifikasi Gate 10 |
| Laporan bertema Gate 10/keamanan 2026-11-15: `RevCore SECURITY AUDIT 151126.md`, `RevCore HARDENING VERIFICATION 151126.md`, `RevCore POST-INSTALL INTEGRITY VERIFICATION 151126.md`, `RevCore BUGFIX ASSET INVENTORY 151126.md` | Seluruh isinya membahas interno Gate 10 |
| Hasil JUnit paket `com.revcore.android.security.*` (20 file XML) | Nama tes keamanan internal Gate 10; hasil agregat tetap dilaporkan di laporan tes |

Selain itu, pada salinan yang diunggah dilakukan redaksi:

- Nama/alias/berkas kunci signing dan lokasi folder kunci → ditandai "tidak dipublikasikan".
- Nama berkas trust record, alias keystore trust, dan marker instalasi internal → ditandai "tidak dipublikasikan".
- Nama variabel environment kata sandi signing → ditandai "env var lokal".
- Rincian algoritma trust record → ditandai "detail tidak dipublikasikan".

Tidak ada kata sandi, isi keystore, materi kunci, atau hash isi kunci di folder ini. Pernyataan "tidak ada `.p12`/`.jks`/`.key` di dalam APK" dipertahankan karena hanya menegaskan ketiadaan, bukan mengungkap materi.

## 3. Label bukti

- **IMPLEMENTED** — ada di source, belum tentu diverifikasi.
- **TESTED (unit)** — lulus unit test JUnit; bukan bukti perilaku di perangkat.
- **VERIFIED (statis)** — diperiksa langsung pada APK/berkas hasil build.
- **DEVICE_UNTESTED** — belum diuji pada perangkat/ECU fisik.

Angka tes pada laporan: 55 suite, 457 tes, 0 gagal/error (hasil saat laporan dibuat). Ini tidak boleh dibaca sebagai validasi kendaraan.

## 4. Fakta kunci paket migrasi

- Paket migrasi: 3.631 file (di luar `SHA256SUMS.txt`), 957 MB; verifikasi SHA-256 3.631/3.631 OK.
- APK resmi 161126: 39.689.642 B, SHA-256 `b2b6c5bd…7432`, v2+v3 aktif, tidak debuggable, cert pin cocok.
- APK 221126: 39.841.086 B, SHA-256 `cb5892a3…ba556`; APK 201126 adalah build antara.
- Bundle Git: 9 commit, branch `master` dan `rebuild-161126`; tidak ada remote dan tidak ada stash di salinan.

## 5. Batas yang tetap berlaku

- Tidak ada pengujian perangkat fisik, USB-OTG/K-Line, ECU, atau respons O2 nyata di laporan mana pun.
- Runtime Gate 10/Keystore perangkat tidak diuji.
- Write/remap tetap terkunci (`readyForWrite=false`) sampai validasi perangkat.
- Dokumen berstatus titik-waktu; kode dapat berubah setelah laporan ditulis.

## 6. Temuan lintas dokumen

- `delivery/protocol_evidence_matrix.md` masih menulis Keyless RENEW `UNSUPPORTED` (`RENEW_SUPPORTED=false`), sedangkan source current menyatakan `RENEW_SUPPORTED=true` dengan bukti `RECONSTRUCTED_FROM_BINARY`. Dokumen lama tidak diedit; perbedaan dicatat di `04_MASALAH_TERBUKA/UNRESOLVED_AND_LIMITS.md` dan di laporan verifikasi WO.
- Hash APK di `OFFICIAL_RELEASE_221126_REPORT.md` pernah salah; nilai benar `cb5892a3…ba556` (tercatat juga di laporan verifikasi paket).
