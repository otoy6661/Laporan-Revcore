# DATABASE KENDARAAN DAN IDENTITAS ECU

Status: deskriptif — ditulis dari source `app/src/main` pada working copy 2026-11-22, bukan dari laporan delivery.
Berkas terkait: `data/ecu/EcmDatabase.kt`, `data/ecu/EcmModel.kt`, `data/ecu/EcuIdCanonicalizer.kt`, `data/ecu/EcuIdentity.kt`, `data/ecu/ChecksumType.kt`, `data/ecu/TempSensorType.kt`, `assets/ecm_database.json`.

## 1. Isi katalog

| Aset | Isi | Bukti |
|---|---|---|
| `assets/ecm_database.json` | 489 record ECM (Keihin 344, Shindengen 132, sisanya vendor lain) | hitung record dengan PowerShell `ConvertFrom-Json` pada working copy, 2026-11-22 |
| `assets/revcore_profiles.json` | 163 profil MAP (declared 163 / actual 163) | hitung kunci `profiles` |
| `assets/revcore_maps/` | 162 berkas BIN + `manifest.tsv` (header + 162 baris) | isi direktori dan manifest |

`ecm_database.json` adalah salah satu dari tiga aset yang dilindungi Gate 10 (lihat `04_GATE10_SECURITY/`). Database ini **read-only untuk aplikasi**: validator tidak pernah menulis ulang record, hanya melaporkan temuan.

## 2. Skema record (`EcmModel`)

Field utama yang dipakai aplikasi:

- Identitas: `id`, `partNumber`, `ecmId`, `calibrationId`, `protocol`, `pinout`.
- Kendaraan: `bikeName`, `yearRange`, `manufacturer`, `country`, `displacementCc`.
- Perangkat: `mcuType`, `flashSizeKb`, `startOffset`, `defaultBaudRate` (default 10 400).
- Batas putaran: `stockRevLimit`, `maxSafeRevLimit`, `recommendedRevLimit`, `revLimitSource`.
- Sensor dan checksum: `tempSensorType` (ECT default), `checksumType` (default `KEIHIN_16BIT_ADD`), `checksumOffset`.
- Peta: offset/ukuran `revLimiter`, `fuelMap` (rows/cols), `ignitionMap` (rows/cols), `idleTarget`.
- Status: `isVerifiedDiagnostic`, `isVerifiedRemap`, `category`, `statusNotes`, `source`.

Semantik penting:

- `isVerifiedRemap = true` **hanya** membuka jalur tulis remap untuk model tersebut (`TuningScreen` memakai `selectedModel.isVerifiedRemap` sebagai `remapSupported`). Default seluruh katalog adalah `false`; jangan menaikkannya tanpa bukti ECU fisik.
- `isActiveCatalogEligible` mensyaratkan tiga hal: `country == "Indonesia"`, `displacementCc` 1..250 cc, dan tidak ada angka >250 di nama model. Record di luar syarat tetap tersimpan dan tetap dipakai untuk identifikasi ECU — hanya tidak muncul di picker.
- `statusNotes` default: "Data referensi; verifikasi hardware belum dilakukan." Jangan menghapus catatan ini dari record yang belum diverifikasi.

## 3. Pencocokan identitas

Dua lapis, keduanya konservatif:

1. **Normalisasi** (`EcmIdentityMatcher`): hanya membuang separator dan menaikkan huruf. Tidak ada lagi penghapusan "00" di depan; kode menyatakan eksplisit bahwa padding tidak pernah dibuktikan.
2. **Relasi terbatas** (`relation`): `EXACT`, atau `PREFIX`/`SUFFIX` hanya bila nilai lebih pendek ≥ 8 karakter dan selisih panjang ≤ 2 byte (mencerminkan field respons 4/5 byte). Selisih lebih besar atau kecocokan ambigu → tidak diidentifikasi.

| `MatchMethod` | Confidence | Arti |
|---|---|---|
| `DB_ID` | 100 | `id` database cocok persis |
| `PART_NUMBER` | 95 | part number cocok persis |
| `ECM_ID_EXACT` | 90 | ECM ID cocok persis |
| `ECM_ID_PREFIX` | 70 | relasi prefix/suffix terbatas |

Confidence adalah **kekuatan bukti, bukan probabilitas**. Beberapa kandidat yang sama kuat dilaporkan `AMBIGUOUS`, bukan dipilih otomatis.

## 4. Kanonikalisasi ID dari frame (`EcuIdCanonicalizer`)

- Panjang field referensi: 5 byte (488 dari 489 record); yang didukung 5 dan 6 byte (`REFERENCE_FIELD_BYTES`, `SUPPORTED_FIELD_BYTES`).
- Input heksadesimal dengan jumlah karakter ganjil → `malformed-hex`, tidak ada ID yang ditebak.
- Bila himpunan ID database diketahui, kandidat field batas penuh yang cocok persis diprioritaskan; jika tidak, field trailing 5 byte dipakai karena pola respons baseline adalah `[selector echo][ECM ID lengkap]`.
- Byte di luar field **tidak dibuang diam-diam**: `protocolPrefixHex`, `trailingHex` disimpan untuk log/tampilan.
- `EcuIdentity` menyimpan `rawResponseHex`, `payloadHex`, `payloadAscii`, `ecuIdRaw`, `ecuIdNormalized`, `canonicalEcuId`, `canonicalCandidates`, `alternateIdentifiers`, `legacyCandidate` (nilai heuristik lama, hanya untuk pembanding), dan `extraction`.

Aturan turunan: ID mentah dan ID kanonik adalah dua nilai berbeda; yang satu tidak menggantikan yang lain. Jangan "merapikan" salah satunya.

## 5. Validasi katalog (`EcmDatabaseValidator`)

Pemeriksaan yang ada (record tidak pernah dihapus karena temuan):

| Kode | Tingkat | Arti |
|---|---|---|
| `BLANK_IDENTITY` | ERROR | id/part number/ECM ID kosong |
| `DUPLICATE_ID` | ERROR | ID database ganda |
| `DUPLICATE_PART_NUMBER` | WARNING | part number ganda |
| `DUPLICATE_ECM_ID` | WARNING | ECM ID ganda |
| `REV_LIMIT_ORDER_CONFLICT` | WARNING | stock/recommended di atas max safe; nilai tidak dipakai untuk rekomendasi |
| `DISPLACEMENT_OUT_OF_SCOPE` | INFO | record Indonesia di luar 1..250 cc; tetap disimpan, tidak masuk katalog aktif |

`EcmDatabase.dataIssues()` menyimpan hasil validasi terakhir untuk ditampilkan/diinspeksi.

## 6. Batas bukti

- 489 record adalah **data referensi**, bukan hasil verifikasi kendaraan. Yang berstatus terverifikasi hanya record dengan `isVerifiedDiagnostic`/`isVerifiedRemap = true`, dan status itu sendiri belum diverifikasi ECU fisik pada rebuild ini.
- Angka rev limit, offset peta, dan checksum hanya boleh dipakai sesuai `revLimitSource` dan `statusNotes` masing-masing record.
- Katalog Indonesia ≤250 cc berasal dari `ecm_database_indonesia_le250_filtered.json` (arsip riset, read-only).

## 7. Yang tidak boleh dilakukan penerus

- Menambah/mengubah record untuk "melengkapi" data tanpa sumber.
- Menaikkan `isVerifiedRemap`/`isVerifiedDiagnostic` tanpa uji kendaraan.
- Mengubah `ecm_database.json` tanpa memperbarui inventaris Gate 10 melalui mekanisme resmi (lihat `04_GATE10_SECURITY/`).
- Mengganti skema JSON atau nama aset; nama aset adalah bagian dari inventory integritas.
