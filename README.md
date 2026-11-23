# Laporan RevCore — Reference Autopsy (H-Diag + 3D Chart Remap K-Line)

Status: analisis referensi teknis. Bukan implementasi, bukan clone, bukan aplikasi baru.

Tanggal: 2026-11-19
Work order: REVCORE REFERENCE AUTOPSY — BAGIAN I & II
Tahap 2: DEEP AUTOPSY TAHAP 2 / H-Diag 2.1 & 3D Chart Graph Remap K-Line (research only)

Laporan Tahap 1 dipertahankan utuh sebagai riwayat penelitian. Dokumen Tahap 2 menambahkan pendalaman, bukti baru, dan koreksi; koreksi tidak menghapus klaim lama, melainkan dicatat di `STAGE2_CONCLUSION_CORRECTIONS.md`.

## Isi

| Folder | Isi |
|---|---|
| `00_INVENTORY/` | Inventaris 869 file (H-Diag 29, 3D Chart 840) dengan SHA-256 |
| `01_HDIAG/` | Ekstraksi string `HDiag.exe` |
| `03_EVIDENCE/` | Register bukti statis, verifikasi checksum frame, file yang dibatasi |
| `04_FINAL_REPORTS/` | Laporan Tahap 1 (tujuh dokumen) + laporan Tahap 2 |
| `05_ARTIFACTS/` | Artefak pendukung (screenshot lama yang dipertahankan) |
| `06_REPOSITORY/` | Daftar file lama yang diganti + laporan pembaruan repository |
| `07_REVCORE_TEKNIS_2026-11-20/` | Laporan teknis audit RevCore 2026-11-20 (Mode `0x5F`, disconnect, matriks status, build/signing, regresi) |
| `08_REVCORE_REFERENCE_AUDIT_2026-11-20/` | Laporan audit referensi penuh 2026-11-20 (H-Diag, FlashPro Keyless, FTDI K-Line, gap RevCore) |
| `09_REVCORE_MIGRASI_2026-11-23/` | Laporan migrasi, rebuild, tes, verifikasi paket, dan verifikasi kesesuaian work order (Gate 10 & inventaris kunci dikecualikan) |
| `10_REVCORE_PELAKSANAAN_2026-11-23/` | Laporan pelaksanaan Master Work Order: audit penuh, tujuh koreksi UI, implementasi protokol, build APK Release resmi 231126 |
| `11_REVCORE_UI_REFINEMENT_2026-11-23/` | Laporan pelaksanaan Work Order UI Refinement: startup/loading, Keyless ID, struktur Tuning, Live Data/AFR/DTC, build resmi baru (SHA-256 `eb94f159…`) |

## Laporan teknis RevCore (2026-11-20)

- [LAPORAN_TEKNIS_REVCORE_2026-11-20.md](07_REVCORE_TEKNIS_2026-11-20/LAPORAN_TEKNIS_REVCORE_2026-11-20.md) — laporan utama
- [INVESTIGASI_MODE_0x5F.md](07_REVCORE_TEKNIS_2026-11-20/INVESTIGASI_MODE_0x5F.md) — investigasi Mode Mesin `0x5F`
- [INVESTIGASI_DISCONNECT_INTERMITTEN.md](07_REVCORE_TEKNIS_2026-11-20/INVESTIGASI_DISCONNECT_INTERMITTEN.md) — investigasi koneksi putus-putus
- [MATRIKS_STATUS_17_PEKERJAAN.md](07_REVCORE_TEKNIS_2026-11-20/MATRIKS_STATUS_17_PEKERJAAN.md) — matriks status + catatan keterlacakan
- [REGRESI_DAN_BUKTI.md](07_REVCORE_TEKNIS_2026-11-20/REGRESI_DAN_BUKTI.md) — regresi, bukti, batas klaim
- [bukti/](07_REVCORE_TEKNIS_2026-11-20/bukti/) — log build/signing/verifikasi + ringkasan unit test

## Laporan audit referensi penuh (2026-11-20)

Work order: `REVCORE FULL REFERENCE AUTOPSY & FTDI K-LINE COMPATIBILITY AUDIT`
(mode READ-ONLY, tanpa implementasi). Folder laporan Next Development sebelumnya
telah dihapus dan diganti seluruhnya; tidak ada file lama dan baru yang dicampur.

- [REVCORE_FULL_REFERENCE_AUDIT.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/REVCORE_FULL_REFERENCE_AUDIT.md) — laporan induk
- [HDIAG_PC_FULL_CAPABILITY_REPORT.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/HDIAG_PC_FULL_CAPABILITY_REPORT.md) — kapabilitas H-Diag
- [HDIAG_LIVE_DATA_PARAMETER_MATRIX.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/HDIAG_LIVE_DATA_PARAMETER_MATRIX.md) — matriks parameter Live Data H-Diag
- [FLASHPRO_KEYLESS_ID_RESEARCH.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/FLASHPRO_KEYLESS_ID_RESEARCH.md) — alur Keyless ID FlashPro
- [FTDI_KLINE_COMPATIBILITY_REPORT.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/FTDI_KLINE_COMPATIBILITY_REPORT.md) — kelayakan FTDI K-Line
- [REVCORE_IMPLEMENTATION_GAP_ANALYSIS.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/REVCORE_IMPLEMENTATION_GAP_ANALYSIS.md) — audit RevCore + gap matrix
- [RESEARCH_EVIDENCE_INDEX.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/RESEARCH_EVIDENCE_INDEX.md) — indeks sumber dan bukti
- [UNRESOLVED_FINDINGS_AND_RISKS.md](08_REVCORE_REFERENCE_AUDIT_2026-11-20/UNRESOLVED_FINDINGS_AND_RISKS.md) — temuan terbuka dan risiko

## Laporan migrasi dan verifikasi (2026-11-23)

Perintah pemilik: unggah laporan migrasi/rebuild/tes/verifikasi ke folder tanggal.
Cakupan dikecualikan atas keputusan pemilik: Gate 10 dan inventaris kunci privat.

- [README folder](09_REVCORE_MIGRASI_2026-11-23/README.md) — indeks isi dan catatan pengecualian
- [Verifikasi kesesuaian work order](09_REVCORE_MIGRASI_2026-11-23/VERIFIKASI_KESESUAIAN_WO_2026-11-23.md) — status per pasal, gap terbuka
- [Verifikasi paket migrasi](09_REVCORE_MIGRASI_2026-11-23/06_MANIFEST_MIGRASI/BACKUP_VERIFICATION_REPORT.md) — 3.631 file, 3.631/3.631 OK

## Laporan pelaksanaan Master Work Order (2026-11-23)

Work order: `FULL SOURCE AUDIT, UI CORRECTION, PROTOCOL IMPLEMENTATION & OFFICIAL SIGNED RELEASE`.
Laporan pelaksanaan (bukan migrasi). Titik awal Build 221126; APK resmi baru:
Build 231126.

- [Laporan pelaksanaan 24 bagian](10_REVCORE_PELAKSANAAN_2026-11-23/LAPORAN_PELAKSANAAN_MASTER_WORK_ORDER_2026-11-23.md) — laporan induk + tabel status
- [README folder](10_REVCORE_PELAKSANAAN_2026-11-23/README.md) — indeks dan ringkasan hasil
- [Matriks bukti protokol](10_REVCORE_PELAKSANAAN_2026-11-23/BUKTI/protocol_evidence_matrix.md) — 33 baris, 5 kelas bukti
- [Rekap unit test](10_REVCORE_PELAKSANAAN_2026-11-23/BUKTI/unit-test-recap-2026-11-23.txt) — 460 test, 0 gagal
- [Verifikasi APK](10_REVCORE_PELAKSANAAN_2026-11-23/BUKTI/apksigner-verify-231126.txt) — v2/v3, sertifikat resmi

## Laporan UI Refinement & Official Signed Release (2026-11-23)

Work order: `REVCORE UI REFINEMENT & OFFICIAL SIGNED RELEASE`. Titik awal Build 231126
(`2b4122aa…`); APK resmi baru `RevCore build 231126.apk` (`eb94f159…`), build lama diarsipkan.

- [Laporan pelaksanaan](11_REVCORE_UI_REFINEMENT_2026-11-23/LAPORAN_PELAKSANAAN_UI_REFINEMENT_2026-11-23.md) — perubahan A–G, source, test, Gate 10, kendala
- [README folder](11_REVCORE_UI_REFINEMENT_2026-11-23/README.md) — indeks dan ringkasan hasil
- [Rekap unit test](11_REVCORE_UI_REFINEMENT_2026-11-23/BUKTI/unit-test-recap-2026-11-23.txt) — 461 test, 0 gagal
- [Verifikasi APK](11_REVCORE_UI_REFINEMENT_2026-11-23/BUKTI/apksigner-verify.txt) — v2/v3, sertifikat resmi
- [Status build](11_REVCORE_UI_REFINEMENT_2026-11-23/BUKTI/build-status.txt) — SUCCESS 2026-11-23T16:28:38+07:00

## Laporan wajib

### H-Diag
- [HDIAG_AUTOPSY_REPORT.md](04_FINAL_REPORTS/HDIAG_AUTOPSY_REPORT.md)
- [HDIAG_FUNCTION_MAPPING.md](04_FINAL_REPORTS/HDIAG_FUNCTION_MAPPING.md)
- [HDIAG_PROTOCOL_ANALYSIS.md](04_FINAL_REPORTS/HDIAG_PROTOCOL_ANALYSIS.md)
- [HDIAG_REVCORE_REQUIREMENTS_04_10.md](04_FINAL_REPORTS/HDIAG_REVCORE_REQUIREMENTS_04_10.md)

### 3D Chart Graph Remap K-Line
- [REVCORE_SENSOR_READING_ANALYSIS.md](04_FINAL_REPORTS/REVCORE_SENSOR_READING_ANALYSIS.md)
- [REVCORE_REMAP_FUNCTION_ANALYSIS.md](04_FINAL_REPORTS/REVCORE_REMAP_FUNCTION_ANALYSIS.md)

### Ringkasan
- [REVCORE_REFERENCE_ANALYSIS_SUMMARY.md](04_FINAL_REPORTS/REVCORE_REFERENCE_ANALYSIS_SUMMARY.md)

## Laporan Tahap 2 (deep autopsy)

### H-Diag 2.1
- [STAGE2_HDIAG_LIVEDATA_PARSER.md](04_FINAL_REPORTS/STAGE2_HDIAG_LIVEDATA_PARSER.md) — peta baris→parser untuk STF, POM, SS, STS, SSW, FIS, MOD, ALT, IAC

### 3D Chart Graph Remap K-Line
- [STAGE2_3DCHART_SENSOR_PROTOCOL.md](04_FINAL_REPORTS/STAGE2_3DCHART_SENSOR_PROTOCOL.md) — sensor, endianness, nilai basi, validasi protokol
- [STAGE2_3DCHART_REMAP_DEEPDIVE.md](04_FINAL_REPORTS/STAGE2_3DCHART_REMAP_DEEPDIVE.md) — keluarga ECU, checksum, blok, ACK, fast write, tanpa read-back

### Sintesis
- [STAGE2_CONCLUSION_CORRECTIONS.md](04_FINAL_REPORTS/STAGE2_CONCLUSION_CORRECTIONS.md) — klaim lama → bukti baru → kesimpulan baru
- [STAGE2_REVCORE_SUMMARY.md](04_FINAL_REPORTS/STAGE2_REVCORE_SUMMARY.md) — terbukti/dugaan/belum diketahui + relevansi RevCore
- [CHANGELOG.md](CHANGELOG.md) — riwayat perubahan repository

### Bukti Tahap 2
- [HDIAG_STAGE2_EVIDENCE.md](03_EVIDENCE/HDIAG_STAGE2_EVIDENCE.md)
- [3DCHART_STAGE2_EVIDENCE.md](03_EVIDENCE/3DCHART_STAGE2_EVIDENCE.md)
- [PROTOCOL_FRAME_VALIDATION_STAGE2.csv](03_EVIDENCE/PROTOCOL_FRAME_VALIDATION_STAGE2.csv) — 45 frame, semua PASS

Catatan supersede (tanpa menghapus Tahap 1):
- Nomor baris H-Diag pada Tahap 1 digantikan tabel 50 baris (STF=20 … IAC=32).
- Label slot `0x0D`/`0x14`/`0x1B`/`0x1E` pada Tahap 1 digantikan pemetaan Tahap 2.
- `array9[2]` Tahap 1 ("DUGAAN tableType") kini terbukti byte fungsi `0x71`.
- "Seed/key berbasis BIN" Tahap 1 dikoreksi: seed/key adalah challenge runtime ECU.

## Bukti utama

- [HDIAG_STATIC_EVIDENCE.md](03_EVIDENCE/HDIAG_STATIC_EVIDENCE.md)
- [3DCHART_SOURCE_EVIDENCE.md](03_EVIDENCE/3DCHART_SOURCE_EVIDENCE.md)
- [PROTOCOL_FRAME_VALIDATION.csv](03_EVIDENCE/PROTOCOL_FRAME_VALIDATION.csv)
- [UNREADABLE_OR_BLOCKED.md](03_EVIDENCE/UNREADABLE_OR_BLOCKED.md)

## Tingkat bukti

| Label | Arti |
|---|---|
| FAKTA / SOURCE VERIFIED | Terbaca langsung dari string, konstanta, kode, atau aritmetika |
| REKONSTRUKSI | Diturunkan dari disassembly; bukan source |
| DUGAAN | Indikasi kuat, belum terbukti |
| SIMULASI | Hasil simulasi (tidak ada dalam pekerjaan ini) |
| UJI PERANGKAT | Pengujian perangkat fisik (tidak ada) |
| UJI ECU | Pengujian ECU nyata (tidak ada) |
| TIDAK DITEMUKAN | Tidak ada bukti |
| BELUM DIUJI | Ada implementasi, belum diverifikasi |

Prinsip: **IMPLEMENTED ≠ VERIFIED**. Tidak ada klaim keberhasilan ECU.

## Batas publikasi

Tidak dipublikasikan:

- `login.ini` atau kode lisensi.
- Key/IV AES lisensi.
- `keygen.exe` / alat bypass lisensi.
- Binary H-Diag, DLL Qt/driver, atau source lengkap pihak ketiga.
- Credential, token, atau data pribadi.

Yang dimuat hanya temuan analisis, potongan kode pendek sebagai bukti, tabel protokol, dan hash.

## Catatan penting untuk RevCore

1. Format frame dan checksum K-line terbukti dan dapat dipakai.
2. Formula sensor berasal dari source 3D Chart, belum diuji ECU.
3. H-Diag tidak membuktikan perintah/byte per parameter; jangan menganggap nama di UI sebagai dukungan penuh.
4. Altitude Mode: hanya pembacaan yang ditemukan; set Mode 1–4 **tidak terbukti**.
5. Remap: tidak ada read-back verification pada aplikasi referensi; verifikasi pasca-tulis adalah kandidat untuk RevCore (belum dibandingkan dengan source RevCore).

## Sanitasi & manifest

- [PUBLICATION_SANITIZATION.md](PUBLICATION_SANITIZATION.md)
- [PUBLICATION_MANIFEST.csv](PUBLICATION_MANIFEST.csv)
