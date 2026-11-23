# MAP, TUNING, DAN REMAP

Status: deskriptif dari source working copy 2026-11-22. Berkas: `data/maps/*`, `core/remap/*`, `ui/screens/TuningScreen.kt`, `ui/tuning/TuningState.kt`, `assets/revcore_maps/`, `assets/revcore_profiles.json`.

## 1. Dua jalur tulis yang berbeda — jangan dicampur

| Jalur | Apa yang ditulis | Status bukti |
|---|---|---|
| MAP Studio (offline) | Berkas `.map` lokal di penyimpanan aplikasi; **tidak ada frame ke ECU** | Dekripsi/enkripsi dan parser direkonstruksi dari biner; format terbukti secara lokal lewat round-trip |
| Remap ROM lewat K-Line | Flash ECU melalui urutan unlock/erase/write/exit hasil dekompilasi | `WRITE_IMPLEMENTED = true`, `WRITE_HARDWARE_VALIDATED = false`; belum ada uji ECU fisik |

`FlashEngine` (state machine default-deny) adalah jalur ketiga yang **tidak pernah** mengaktifkan tulis: `FlashCapabilities.WRITE_PROTOCOL_PROVEN`, `READBACK_PROVEN`, `RECOVERY_PROVEN`, `BENCH_TEST_PERFORMED` semuanya `false`. Jangan "menyatukan" ketiga jalur ini.

## 2. Aset MAP

- `assets/revcore_maps/manifest.tsv` — header `fileName, bytes, sha256, profileId, profileCandidates, matchMethod, status, note`, 162 baris data.
- 162 berkas BIN terenkripsi milik pemilik; disimpan apa adanya.
- `assets/revcore_profiles.json` — 163 profil VomConfig. Profil dengan `index <= 0`, ECM ID kosong/"0", atau resource kosong dianggap placeholder dan tidak ditampilkan (`isPlaceholder`).
- Nama aset (`revcore_maps`, `manifest.tsv`, `revcore_profiles.json`) dan skema kunci JSON (`tpsh`, `rpmh`, `feul1`, …) dipertahankan; semuanya tercatat di inventory Gate 10.

### Pencocokan nama berkas → profil (`OriginalMapMatcher`)

Urutan deterministik: (1) resourceName sama persis setelah normalisasi → `RESOURCE_NAME`; (2) tepat satu label profil terkandung di nama berkas → `LABEL_IN_FILENAME`; (3) lebih dari satu kandidat → `AMBIGUOUS`, tidak ada yang dipilih; (4) tidak ada bukti → `NONE`. Tidak ada "profil terdekat".

### Status manifest

`READY` (boleh dibuka), `AMBIGUOUS_PROFILE` (tampil sebagai ambigu), `UNRECOGNIZED` (hanya mentah). Baris manifest yang rusak dilewati, bukan ditebak.

## 3. Kontainer `.map` / `.enc` (`MapCrypto`)

- AES-128-CBC, PKCS#5, key `0123456789ABCDEF` (UTF-8), IV tetap `ABCDEF0123456789` — direproduksi dari `BinLoader`.
- Panjang ciphertext = payload + 16, selalu satu blok padding penuh.
- **Tidak ada MAC, signature, atau checksum** di kontainer. Karena itu dekripsi tidak membuktikan keaslian berkas; validasi hanya membuktikan struktur dan kecocokan profil.
- Kegagalan padding divalidasi byte demi byte; buffer hasil dekripsi parsial tidak pernah dikembalikan.
- Batas input 64 MiB. Ekstensi selain `.map`/`.enc` dianggap plain.
- Enkripsi ekspor deterministik (padding 1..16 mengikuti panjang).

## 4. Katalog kerja (`MapCatalog`)

- Impor **manual**: pengguna memilih profil eksplisit (`profileId`), tidak ada best-match.
- Salinan asli disimpan terenkripsi dan tidak berubah; setiap edit membuka salinan kerja baru.
- Setiap entri menyimpan `cipherSha256`, `payloadSha256`, ukuran, profil, status validasi, waktu impor.
- Entri hasil edit diberi akhiran "(salinan edit)" dan `parentId`.
- Undo/redo dan reset tersedia di `EditableMap`; `exportEncrypted()` hanya menulis salinan lokal.

## 5. Parser dan validator

`MapParser` mereproduksi `parseSection`/`BinParser`: offset absolut, word 1/2 byte, endian eksplisit, codec (U8/U16, RPM axis, limiter, ignition 255/alt, IAP axis), pemeriksaan jendela ketat. Section yang jendelanya tidak valid menjadi `ParsedSection.Unknown` dan **tidak** dirender dengan nol palsu.

`MapValidator`:

- Ukuran image yang dikenal: 48 KiB, 64 KiB, 256 KiB, 384 KiB.
- Status: `VERIFIED_COMPATIBLE_FOR_OFFLINE_PROFILE`, `PROFILE_MATCH_ONLY`, `INCOMPLETE_EVIDENCE`, `INCOMPATIBLE`, `FLASHING_BLOCKED`.
- Offset bertumpuk, section dengan `HYPOTHESIS`, atau `UNKNOWN` selalu dilaporkan.
- `FLASHING_BLOCKED` tidak pernah menjadi izin flashing.

Tingkat bukti definisi (`MapDefinitionConfidence`): `CONFIRMED` = mengikuti aturan hasil dekompilasi dan seluruh rentang byte diperiksa; `HYPOTHESIS`; `UNKNOWN` = hanya baca.

## 6. Remap ROM lewat K-Line (`core/remap`)

Parameter (`RemapCodec`):

| Konstanta | Nilai |
|---|---|
| `BAUD_RATE` | 10 400 |
| `FAST_BAUD_RATE` | 62 500 |
| `BLOCK_SIZE` | 128 byte |
| `READ_RETRIES` | 3 |
| Jeda balasan tulis / retry | 5 ms / 10 ms |
| Jeda balasan baca / gap retry | 5 ms / 10 ms |

Alur UI (`TuningScreen` kartu Remap Execution): BACA ROM → VERIFIKASI → SIMPAN BACAAN → TULIS BIN → JUMLAH FLASH → reset. Tulis memakai dialog konfirmasi ketik; setelah tulis host langsung membaca balik rentang yang sama dan **hanya read-back yang cocok** dilaporkan sebagai tulis terverifikasi.

Guard (`RemapExecutionGuard`):

- Batas operasi host 0x100..512 KiB (batas host, bukan klaim geometri ECU).
- Rute vendor hanya Keihin (H99B/KWP) dan Shindengen (H99B); keluarga lain ditolak, bukan dipaksa.
- Penolakan: `BUSY`, `ADAPTER_CLOSED`, `MODEL_UNKNOWN`, `MODEL_NOT_REMAP_VERIFIED`, `VENDOR_UNKNOWN`, `ROM_MISSING`, `FLASH_SIZE_UNKNOWN`, `SIZE_MISMATCH`, `SIZE_OUT_OF_RANGE`, `CHECKSUM_UNKNOWN`, `CONFIRMATION_REQUIRED`.
- Keihin menolak tulis bila lokasi checksum tidak diketahui pada image tersebut.

`RemapWorkflow` menghitung langkah dan gate; `readyForWrite` mensyaratkan `WRITE_HARDWARE_VALIDATED` (false), jadi secara definisi selalu terkunci walaupun jalur eksekusi dengan konfirmasi tetap ada. Kartu UI menyatakan bukti `RECONSTRUCTED_FROM_BINARY; belum ada uji ECU fisik`.

## 7. Batas dan larangan

- Jangan mengklaim tulis BIN berhasil berdasarkan ACK; hanya read-back yang cocok adalah bukti.
- Jangan menaikkan `WRITE_HARDWARE_VALIDATED`, `isVerifiedRemap`, atau capability `FlashEngine` tanpa uji kendaraan yang terdokumentasi.
- Jangan mengubah key/IV, skema JSON profil, nama aset, atau header manifest tanpa mekanisme resmi (aset dilindungi Gate 10).
- Jangan menambah seleksi profil otomatis dari nama berkas; ambiguitas harus tetap terlihat.
- Belum ada uji pemulihan setelah listrik terputus saat tulis; `RECOVERY_REQUIRED` adalah status, bukan prosedur yang terbukti.
