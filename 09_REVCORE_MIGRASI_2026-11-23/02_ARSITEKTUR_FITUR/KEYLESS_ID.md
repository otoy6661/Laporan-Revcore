# KEYLESS ID (IMMOBILIZER)

Status: deskriptif dari source working copy 2026-11-22. Berkas: `core/keyless/*`, `ui/screens/KeylessScreen.kt`, `ui/keyless/KeylessUiState.kt`.

## 1. Urutan UI yang ditetapkan pemilik

`KeylessScreen` merender lima bagian bernomor tetap:

1. `1` KENDARAAN TERIDENTIFIKASI
2. `2` ECM (KEIHIN / SHINDENGEN)
3. `3` BACA KEY ID
4. `4` RESET KEY ID
5. `5` RENEW KEY ID

Tidak ada pemilihan model manual. Profil berasal dari `KeylessProfileResolver` berdasarkan **part number** ECU yang benar-benar terbaca. Tabel firmware berisi 27 record (Shindengen 23, Keihin 4), semuanya menyimpan part number, bukan ECM ID; ECM ID tidak pernah "diarang" untuk record Keyless.

## 2. Prasyarat aktivasi

`KeylessUiState.canRunMaintenance` mensyaratkan **semua**:

- adapter terhubung,
- tidak ada operasi Keyless lain yang berjalan,
- profil berhasil di-resolve (`matched != null`),
- `readResultValid == true` — hasil BACA KEY ID masih valid untuk profil saat ini,
- `report.success == true`.

`readResultValid` dikosongkan saat adapter terputus atau profil berubah, sehingga ID basi tidak pernah ditampilkan sebagai bacaan terkini. RESET/RENEW juga meminta konfirmasi eksplisit (dialog).

## 3. Resolusi profil (`KeylessProfileResolver`)

| Metode | Arti |
|---|---|
| `EXACT_PART_NUMBER` | part number sama persis setelah normalisasi |
| `VARIANT_PART_NUMBER` | record firmware bertag negara (mis. `…-A71 IN01`) dan part number ECM sama dengan basis + maksimal 2 karakter |
| `SHARED_PROFILE_FOR_CANDIDATES` | identitas ECU parsial/ambigu, tetapi semua kandidat menunjuk satu profil yang sama |

Dua profil **berbeda** yang cocok untuk kendaraan yang sama → `Ambiguous`, tidak ada profil yang dipilih. Kendaraan di luar tabel → tidak ada profil, dan engine menolak membaca.

## 4. BACA KEY ID (`KeylessReadEngine` + `KeylessFrameCodec`)

Alur: test-present → start communication → probe/security access keluarga → baca 256 alamat EEPROM (0x00..0xFF) dengan frame keluarga yang terdeteksi → decode.

Frame kunci (direproduksi dari referensi C# `SMK.cs`, nilai wire dipertahankan):

| Frame | Bytes |
|---|---|
| Test present | `FE 04 72 8C` |
| Start communication | `72 05 00 F0 99` |
| Keihin security request / key | `27 0B E0 48 65 6C 6C 6F 48 6F 43` / `27 0B E0 77 41 72 65 59 6F 75 22` |
| Shindengen security request / key | `91 91 0D DF 9E 8D 9A 86 90 8A 8C 9B 88` / `91 91 0D DF 92 9E 86 96 8B 8D 86 C0 6A` |
| Keihin read | `82 82 10 06 addr chk` |
| Shindengen read | `91 91 07 40 addr 00 chk` (checksum hanya `07+40+addr`) |

Ukuran balasan minimum: Keihin 0x0C byte, Shindengen 0x0D byte; indeks byte data masing-masing 10/11 dan 11/12. Balasan yang terlalu pendek berarti "tidak terbaca", bukan nol.

Decode menjadi ID 9 digit (`KeylessIdDecoder`):

- `decodeFirmwareMode0`: `b1 | b2<<8 | b3<<16 | b4<<24`, `%09lu`.
- `decodeFirmwareMode1` (SCOOPY): `(b2<<8|b1)*0x4000 + 0xE0000000 + ((b3|b4<<8) xor 0x8000)` aritmetika 32-bit unsigned.
- `decodeLegacyPcxStyle` disimpan hanya sebagai pembanding; nilai firmware adalah yang utama.
- Tidak ada transform yang berstatus `FIELD_VERIFIED`.

## 5. RESET dan RENEW

Keduanya adalah **penulisan nol** area kunci, bukan pemasangan kunci baru. Firmware menamai rutinnya "Renew", tetapi frame pemasangan kunci baru tidak ada di biner.

| | Keihin | Shindengen |
|---|---|---|
| Area | 4 byte image 0x46..0x49 (halaman EEPROM 0x23..0x24) | 176 alamat 0x00..0xAF |
| Frame tulis | `82 82 14 08 page low high chk`, `page = index >> 1`, checksum `-(page + 0x20 + low + high)` | `91 91 09 41 addr 00 00 00 chk`, checksum `-(09+41+addr)` |
| Timeout balasan | 500 ms | 400 ms |
| Jeda antar frame | 120 ms | 5 ms |
| Toleransi gagal | 0 frame gagal | < 10 frame gagal |

Prasyarat engine (`KeylessResetEngine`): konfirmasi eksplisit, adapter terbuka, model didukung, **pembacaan prasyarat penuh sukses**, dan keluarga frame yang terdeteksi sama dengan keluarga model. Keihin membutuhkan `KEIHIN_READ_PRECONDITION_BYTES = 0x4A` byte terbaca; pasangan halaman 0x23/0x24 yang tidak terbaca adalah penolakan, bukan pengganti nol.

ACK bukan bukti keberhasilan: `isAcknowledged` mensyaratkan balasan ≥ 5 byte dan bukan sekadar local echo. Setelah menulis, engine **membaca balik** area target; hanya semua alamat menjawab dan bernilai nol yang menghasilkan `SUCCESS`.

## 6. Taksonomi hasil

`KeylessProcessOutcome`: `SUCCESS`, `FAILED`, `PARTIAL`, `UNKNOWN`, `RECOVERY_REQUIRED`. `PARTIAL`/`UNKNOWN`/`RECOVERY_REQUIRED` adalah hasil kelas satu karena penulisan area kunci tidak atomik; kehilangan link di tengah tidak boleh dilaporkan sebagai gagal-tanpa-efek atau sukses.

Status UI per langkah (`KeylessStepStatus`): BACA memakai WAITING/FAILED/SUCCESS; RESET dan RENEW memakai NOT_AVAILABLE/FAILED/SUCCESS. `resetReportByAction` menjaga status RESET dan RENEW terpisah.

## 7. Batas bukti dan larangan

- `RESET_SUPPORTED = true`, `RENEW_SUPPORTED = true`, `RESET_EVIDENCE = RECONSTRUCTED_FROM_BINARY`; belum ada eksekusi pada ECU fisik.
- Dokumen delivery lama (`change_summary.md`, `protocol_evidence_matrix.md`, `unresolved_items.md`) masih menyebut `RENEW_SUPPORTED=false`. Source commit `329405a` adalah keadaan sebenarnya; dokumen lama tidak diubah, selisih dicatat di `10_KNOWN_ISSUES/`.
- RENEW tidak boleh dipresentasikan sebagai "pasang kunci baru"; tidak ada frame seperti itu.
- Jangan mengubah frame, checksum, indeks, timeout, atau urutan langkah tanpa bukti firmware baru dan persetujuan Otoy.
- Jangan menurunkan syarat pembacaan penuh/konfirmasi/verifikasi read-back.
- Pembacaan EEPROM penuh adalah prasyarat keamanan, bukan formalitas; melewatinya bisa menulis area kunci yang salah.
