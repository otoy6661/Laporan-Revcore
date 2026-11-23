# Changelog

Semua perubahan laporan pada repository ini. Format mengikuti Keep a Changelog secara ringkas; tanggal dalam ISO 8601.

## [RevCore Master Work Order Execution] — 2026-11-23

Laporan pelaksanaan Master Work Order penuh (audit source, koreksi UI, implementasi
protokol, build APK Release resmi). Bukan laporan migrasi; arsip migrasi `09_*`
tidak diubah dan tidak dihapus.

### Added
- `10_REVCORE_PELAKSANAAN_2026-11-23/` — laporan pelaksanaan 24 bagian + tabel
  status 17 baris, README indeks, dan bukti (`BUKTI/`): matriks bukti protokol,
  rekap unit test (460 test, 55 suite, 0 gagal/error), status marker build,
  SHA-256 APK, hasil `apksigner verify`, `aapt2 badging`, inventaris aset APK.

### Changed
- `README.md` — tabel isi dan bagian tautan laporan pelaksanaan 2026-11-23.

### Notes
- APK Release resmi: `RevCore build 231126.apk`, 39.841.086 byte, SHA-256
  `2b4122aa18986417bf34b9b37887dfa391058262d3c2775c655d1cae64647ae9`,
  sertifikat `c649cb4f…18d098` (sama dengan Build 221126). APK tidak diunggah ke
  repository laporan.
- Tidak ada laporan lama yang dihapus: tidak ada laporan UI/implementasi lama di
  repository ini yang secara langsung digantikan; audit dan migrasi historis
  dipertahankan.

## [RevCore Migration & Verification] — 2026-11-23

Perintah pemilik: unggah laporan migrasi, rebuild, tes, dan verifikasi ke folder
tanggal. Cakupan yang dikecualikan: Gate 10 (`04_GATE10_SECURITY`) dan inventaris
kunci privat (`05_PRIVATE_KEY_INVENTORY`), termasuk skrip/laporan bertema Gate 10.

### Added
- `09_REVCORE_MIGRASI_2026-11-23/` — 115 file: README indeks, verifikasi kesesuaian
  work order, riwayat proyek, arsitektur/fitur, laporan build/signing/tes, log build,
  hasil JUnit baseline/current, masalah terbuka, indeks riset, manifest migrasi,
  dan catatan Git.
- `09_REVCORE_MIGRASI_2026-11-23/VERIFIKASI_KESESUAIAN_WO_2026-11-23.md` — verifikasi
  baca-saja per pasal A–J, termasuk tiga gap (D1, D3, C1) dan satu dokumen kedaluwarsa (C4).

### Changed
- `README.md` — tabel isi dan tautan folder 2026-11-23.
- `PUBLICATION_MANIFEST.csv` — entri hash untuk file baru.

### Security
- Salinan laporan disanitasi: nama/alias/lokasi kunci signing, nama berkas trust
  record, dan nama env var kata sandi diganti penanda "tidak dipublikasikan".
- Tidak ada kata sandi, isi kunci, atau materi rahasia di file baru.

## [RevCore Full Reference Audit] — 2026-11-20

Work order: `REVCORE FULL REFERENCE AUTOPSY & FTDI K-LINE COMPATIBILITY AUDIT`.
Mode READ-ONLY: tidak ada perubahan source RevCore, tidak ada implementasi baru,
tidak ada ZIP/publikasi APK, tidak ada klaim uji ECU (kelas D kosong).

### Changed
- Folder `08_REVCORE_NEXT_DEV_2026-11-20/` dihapus seluruhnya dan diganti
  `08_REVCORE_REFERENCE_AUDIT_2026-11-20/`; file lama dan baru tidak dicampur.
- `README.md` diperbarui ke set laporan audit referensi penuh.

### Added
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/REVCORE_FULL_REFERENCE_AUDIT.md` — laporan induk.
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/HDIAG_PC_FULL_CAPABILITY_REPORT.md` — kapabilitas H-Diag.
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/HDIAG_LIVE_DATA_PARAMETER_MATRIX.md` — matriks parameter H-Diag.
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/FLASHPRO_KEYLESS_ID_RESEARCH.md` — alur Keyless ID.
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/FTDI_KLINE_COMPATIBILITY_REPORT.md` — kelayakan FTDI.
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/REVCORE_IMPLEMENTATION_GAP_ANALYSIS.md` — gap matrix RevCore.
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/RESEARCH_EVIDENCE_INDEX.md` — indeks bukti.
- `08_REVCORE_REFERENCE_AUDIT_2026-11-20/UNRESOLVED_FINDINGS_AND_RISKS.md` — temuan terbuka.

### Evidence
- Delapan dokumen wajib work order; hash sumber tercatat di `RESEARCH_EVIDENCE_INDEX.md`.
- Status ECU tetap UNVERIFIED; tidak ada upgrade status tanpa uji perangkat.

## [RevCore Next Development] — 2026-11-20

Work order: `WORK ORDER — REVCORE NEXT DEVELOPMENT`. Implementasi lokal pada
source RevCore ditambah laporan bukti. Tidak ada APK/ZIP publik, tidak ada
perubahan identitas/branding, tidak ada perubahan Gate 10.

### Added
- `08_REVCORE_NEXT_DEV_2026-11-20/RINGKASAN_WORK_ORDER_2026-11-20.md` — status, perubahan, bukti, risiko.
- `08_REVCORE_NEXT_DEV_2026-11-20/LAPORAN_PEKERJAAN_1_KEYLESS_ID.md` — investigasi Keyless: tidak ada transaksi security di log, celah logging, perbaikan logging.
- `08_REVCORE_NEXT_DEV_2026-11-20/LAPORAN_PEKERJAAN_2_LIVE_MONITOR.md` — matriks 19 parameter versus implementasi RevCore dan H-Diag.
- `08_REVCORE_NEXT_DEV_2026-11-20/LAPORAN_PEKERJAAN_3_LOG_FEATURE.md` — filter error, Open Location, Share lampiran file.
- `08_REVCORE_NEXT_DEV_2026-11-20/LAPORAN_PEKERJAAN_4_KESTABILAN_ADAPTER.md` — audit lifecycle USB/serial + perbaikan idempotensi close.
- `08_REVCORE_NEXT_DEV_2026-11-20/LAPORAN_PEKERJAAN_5_SCREEN_AWAKE.md` — window flag tanpa wakelock.
- `08_REVCORE_NEXT_DEV_2026-11-20/bukti/` — log unit test, build debug/release-Kotlin, verifikasi APK/manifest.

### Changed (source RevCore, build lokal)
- Predikat error tunggal + riwayat error 500 entri; ekspor hex lengkap dan bagian `ERROR HISTORY`.
- Filter ERROR memakai predikat yang sama dengan penghitung error.
- Share menulis file `.txt` ke cache dan memakai FileProvider + URI grant.
- Open Location memeriksa izin, folder, `resolveActivity`, dan exception.
- Frame Keyless TX/RX/ERROR/STATE masuk ke log ekspor.
- `isOpen` volatile; `close()` synchronized/idempoten.
- Keep-screen-on selama Activity terlihat.

### Evidence
- 335 unit test, 0 gagal.
- `assembleDebug` dan `compileReleaseKotlin` berhasil.
- APK debug SHA-256 `1d1566ba97b3a38b28751d36141fec96bdcff19e3742ced6cd33261c953acf37`.
- Tidak ada pengujian hardware; pekerjaan berstatus PARTIAL.

### Preserved
- Riwayat Git, laporan Tahap 1/Tahap 2, identitas otoy, dan semua konfigurasi kendaraan.
- Tidak ada signing key/credential yang diunggah.

## [Laporan Teknis RevCore] — 2026-11-20

Work order: `WORK ORDER — LAPORAN TEKNIS REVCORE` (upload laporan hasil
pekerjaan dan investigasi bug). Dokumentasi saja: tidak ada source RevCore yang
diubah, tidak ada Gate 10 yang diubah, tidak ada rahasia signing.

### Added
- `07_REVCORE_TEKNIS_2026-11-20/LAPORAN_TEKNIS_REVCORE_2026-11-20.md` — laporan utama (mode `0x5F`, disconnect, status, build/signing, regresi).
- `07_REVCORE_TEKNIS_2026-11-20/INVESTIGASI_MODE_0x5F.md` — bukti lokasi tampilan, offset dat17, decoder `1..4`, perbandingan dengan referensi 3D Chart.
- `07_REVCORE_TEKNIS_2026-11-20/INVESTIGASI_DISCONNECT_INTERMITTEN.md` — rantai USB/K-Line, timeout, backoff, state `SUSPECTED_LOSS`.
- `07_REVCORE_TEKNIS_2026-11-20/MATRIKS_STATUS_17_PEKERJAAN.md` — tabel status jujur + catatan bahwa daftar asli tidak ditemukan.
- `07_REVCORE_TEKNIS_2026-11-20/REGRESI_DAN_BUKTI.md` — regresi, batas klaim, prosedur reproduksi.
- `07_REVCORE_TEKNIS_2026-11-20/bukti/` — log fail-closed signing, log unit test, verifikasi APK resmi (161126 & Keyless), ringkasan 330 test, SHA-256.

### Changed
- `README.md` — indeks folder 07 dan tautan laporan baru.
- `PUBLICATION_MANIFEST.csv` — hash SHA-256 diperbarui dan entri folder 07 ditambahkan.

### Preserved
- Seluruh laporan Tahap 1/Tahap 2 dan riwayat Git tidak diubah.
- Tidak ada APK/ZIP yang diunggah; hanya hash dan hasil verifikasi.

## [Tahap 2] — 2026-11-19

Work order: DEEP AUTOPSY TAHAP 2 / H-Diag 2.1 & 3D Chart Graph Remap K-Line. Research only: tidak ada implementasi, patch, atau perubahan RevCore.

### Added
- `03_EVIDENCE/PROTOCOL_FRAME_VALIDATION_STAGE2.csv` — audit ulang 45 frame; semua checksum PASS.
- `03_EVIDENCE/HDIAG_STAGE2_EVIDENCE.md` — tabel 50 baris, dispatcher token-count, mekanika `setRow`, 48 call site, tabel per tipe ECU, jalur DTC.
- `03_EVIDENCE/3DCHART_STAGE2_EVIDENCE.md` — `InitFtdiIso`, `SendCommand` (echo/panjang/checksum/timeout), polling, pemetaan dat.
- `04_FINAL_REPORTS/STAGE2_HDIAG_LIVEDATA_PARSER.md` — sembilan parameter wajib: baris, jalur balasan, slot, batasan.
- `04_FINAL_REPORTS/STAGE2_3DCHART_SENSOR_PROTOCOL.md` — sensor per dat, endianness, nilai basi, validasi protokol.
- `04_FINAL_REPORTS/STAGE2_3DCHART_REMAP_DEEPDIVE.md` — signature ECU, checksum, alamat vs indeks blok, ACK, fast write, ketiadaan read-back.
- `04_FINAL_REPORTS/STAGE2_CONCLUSION_CORRECTIONS.md` — 14 koreksi klaim Tahap 1 beserta alasan.
- `04_FINAL_REPORTS/STAGE2_REVCORE_SUMMARY.md` — daftar terbukti/dugaan/unknown/hardware + relevansi kelas 1–4.
- `CHANGELOG.md` — file ini.

### Changed
- `README.md` — indeks Tahap 2, catatan supersede, dan tautan bukti baru.
- `PUBLICATION_MANIFEST.csv` — hash SHA-256 seluruh file setelah pembaruan.

### Preserved
- Seluruh laporan dan bukti Tahap 1 tidak diubah dan tetap menjadi riwayat penelitian.
- Identitas, kredit, dan riwayat commit Otoy tidak diubah; tidak ada force push.

### Corrections (ringkas)
- Nomor baris H-Diag: 30 baris (STF=0) → 50 baris (RPM=0, STF=20, IAC=32).
- Slot `0x0C` dikoreksi dari IAC ke O2; slot `0x14` dari TPSV2 ke STF; slot `0x0F/0x1B/0x1E` dari EVA/EGCV/tak dikenal ke BAS/ALT/MIL.
- `array9[2]` dari "DUGAAN tableType" menjadi byte fungsi `0x71` (A).
- Timeout: dari "tampil nol" menjadi "nilai lama bertahan" saat `b[1]==5`.
- Seed/key: dari "berbasis BIN" menjadi challenge runtime ECU.
- Fast-write frame: dari DUGAAN menjadi Source Verified (khusus `KEIHINOTHER`).
- "Bug alamat" remap dibatalkan: `startOffset/16` (alamat) vs `/128` (indeks) adalah desain.

## [Tahap 1] — 2026-11-19

Work order: REVCORE REFERENCE AUTOPSY — BAGIAN I & II.

### Added
- Struktur baru repository: inventory 869 file, ekstraksi string H-Diag, register bukti, tujuh laporan wajib, artefak lama.
- `03_EVIDENCE/PROTOCOL_FRAME_VALIDATION.csv` — 30 frame.
- `PUBLICATION_MANIFEST.csv`, `PUBLICATION_SANITIZATION.md`.
- `06_REPOSITORY/REMOVED_OLD_FILES.txt`, `06_REPOSITORY/REPOSITORY_UPDATE_REPORT.md`.

### Preserved
- Tiga screenshot unggahan pemilik dari commit `9b7037e` dipindahkan ke `05_ARTIFACTS/legacy_screens/`.
- Riwayat 136 file lama tetap tersedia melalui commit tersebut.
