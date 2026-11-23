# LIVE DATA, AFR, DTC, DAN LOG

Status: deskriptif dari source working copy 2026-11-22. Berkas: `core/comm/LiveDataProfile.kt`, `HondaEcuProfiles.kt`, `HondaProtocolEngine.kt`, `AfrEstimator.kt`, `LivePollPolicy.kt`, `core/protocol/CommunicationSniffer.kt`, `HondaDtcDatabase.kt`, `ui/livedata/LiveDataPresentation.kt`, `ui/screens/LiveDataScreen.kt`, `DiagnosticsScreen.kt`, `LogScreen.kt`, `core/storage/RevCoreStorage.kt`.

## 1. Live Data — aturan yang ditetapkan pemilik

1. Sebelum adapter terhubung: **tidak ada tabel kosong** dan tidak ada pesan debug. `LiveDataPresentation.pane` mengembalikan `CONNECT_VEHICLE`.
2. Setelah terhubung tetapi belum ada data: `WAITING_FOR_DATA`, bukan angka nol.
3. Setelah sinkronisasi: hanya parameter yang benar-benar tervalidasi dari frame yang tampil.
4. Parameter tambahan yang hanya terkirim saat mesin hidup boleh muncul belakangan; parameter sebelumnya **tidak boleh hilang** (`retainPreviousReadings` menyimpan nilai lama sebagai STALE dengan timestamp aslinya).
5. Prakiraan AFR hanya tampil bila O2 berstatus VALID/STALE.
6. Pemilihan/polling yang ada tidak boleh diganti; angka atau parameter tidak boleh dikarang.

## 2. Pipeline

`parseLiveFrame` (frame → sensor) → `LiveParameterStates.withEngineRunning` → `retainPreviousReadings` → `LiveTelemetry` → `UnifiedLiveTable.build` (hanya sensor yang hadir) → `LiveDataScreen`.

- Tabel dan id profil: `revcore.dat17`, `revcore.dat13`, `revcore.dat11`, `revcore.dat10`, plus ekstensi `revcore.dat16`, `revcore.dat20`, `revcore.datD1`. Label UI: "Live data tabel 0x17 (revcore.dat17)" dst.; label `dat16/dat20/datD1` bertanda "tambahan".
- `LiveParameterState`: mencakup `WAITING_FOR_DATA`, nilai valid, dan `STALE` untuk nilai yang tidak lagi dikirim.
- `LivePollPolicy`: interval minimum 150 ms, maksimum 1000 ms; maksimum 4 kegagalan berturut; backoff 300→3000 ms. Engine melewati ekstensi selama `EXTENSION_SKIP_MS = 15_000` ms setelah kegagalan dan memakai cooldown fallback PID 10 000 ms.
- Balasan yang datang setelah sesi link berubah dibuang (commit `eaa9117`), sehingga nilai tabel lama tidak dipakai untuk sesi baru.

Satu tabel tanpa Role Table: `UnifiedLiveTable` menyatukan tabel dasar dan ekstensi menjadi satu daftar parameter; tidak ada kolom peran (role) yang dipakai sebagai identitas.

## 3. Prakiraan AFR (`AfrEstimator`)

- Sumber: tegangan O2 narrowband dari tabel `dat20` (`72 05 71 20 F8`, byte 4), skala firmware `1 / 50.993698`.
- Rumus: `AFR = 17.700001 − 5.351 × V_O2`, rentang narrowband 0.0–1.1 V.
- Label: "Prakiraan AFR" / "Estimated AFR".
- Peringatan tetap: "Perkiraan host dari tegangan O2 narrowband dat20, bukan AFR terukur. Jangan dipakai untuk tuning."
- Hasil selain `Estimated`: `Unavailable` (O2 tidak ada / bukan angka), `Invalid` (di luar rentang atau status bukan VALID/STALE). Tidak ada clamp diam-diam, tidak ada nilai pengganti.
- `showsAfr` mensyaratkan `o2State == VALID` (STALE masih diterima estimator sebagai perkiraan berlabel).

## 4. DTC (`DiagnosticsScreen`, `HondaDtcDatabase`, `HondaProtocolEngine`)

Pembacaan menghasilkan tiga kemungkinan: `Success(dtcs, format)`, `NoResponse(reason)`, `Unrecognized(rawHex, reason)`. Hanya `Success` yang boleh disimpulkan sebagai status DTC; `NoResponse`/`Unrecognized` secara eksplisit berarti status tidak dapat disimpulkan.

- Parser memerlukan frame ≥ 4 byte, panjang konsisten dengan byte[1], dan checksum valid. Format konten yang didukung: `[echo 0x03][pasangan]`, `[echo 0x00][pasangan]`, `[count][code,sub,…]`, `[code,sub,…]`.
- Pasangan decode = (high byte, low byte). Kode yang dikenal memakai deskripsi database; kode tak dikenal ditampilkan `DTC %02X%02X` dengan judul "belum terverifikasi" — P-code standar tidak dikarang.
- Database dikenal berisi 15 kode (MAP, ECT/EOT, TP, IAT, VSS, injektor, O2, IACV, checksum ECM, bank angle). UI menampilkan `standardCode (MIL #blinkCode)`.
- Hapus DTC: hanya aktif dengan adapter hidup dan tidak ada operasi DTC lain. Status hasil: `SUCCESS_VERIFIED`, `CLEARED_UNVERIFIED`, `RE_DETECTED_ACTIVE_FAULT`, `REJECTED_OR_TIMEOUT`, `UNSUPPORTED_FAMILY`, `NO_ADAPTER`. Keluarga yang frame clear-nya belum direkonstruksi (`dtcClear` kosong) ditolak tanpa mengirim perintah. ACK saja = `CLEARED_UNVERIFIED`; `SUCCESS_VERIFIED` hanya setelah pembacaan ulang tidak menemukan kode.
- Selama baca/hapus DTC, streaming Live Data dihentikan lalu dipulihkan di `finally`.

Tata letak: kartu ringkasan (TIDAK ADA DTC / N DTC AKTIF / belum terbaca / tidak dapat dipastikan), daftar DTC yang bisa di-scroll, dan tombol HAPUS DTC tetap terjangkau di bawah.

## 5. Log komunikasi

Sumber: `CommunicationSniffer` (buffer event) + `LogScreen`.

- Kategori event: `ALL`, `ERROR`, `FTDI`, `KLINE`, `PROTOCOL`, `RX`, `SYSTEM`, `TX`, `USB`.
- TXT: `exportAsText()` memuat header RevCore, ECU target, waktu ekspor, statistik TX/RX/error, konfigurasi port (`10400/9600 8N1 … DTR=1, RTS=1, Latency=2ms`), ringkasan sesi berbasis bukti (status tertinggi, frame valid/invalid, local echo, timeout, retry, USB error, parse failure, init attempts), lalu event.
- CSV: kolom `ID,Timestamp,EpochMs,Category,Stage,ByteCount,HexString,AsciiString,Status,DurationMs,Description,ErrorDetails`.
- Simpan: `Internal Storage/Revcore/Logs` melalui `RevCoreStorage`; bila izin belum ada, status `NeedsPermission` dengan alasan dan jalan ke pengaturan — tidak ada fallback diam-diam ke penyimpanan privat.
- Bagikan: file ditulis ke `cacheDir/share`, dibagikan lewat `FileProvider` + `ACTION_SEND` dengan `FLAG_GRANT_READ_URI_PERMISSION`.
- Buka lokasi: `RevCoreStorage.folderViewIntents` mencoba URI dokumen folder, MIME `resource/folder`, lalu root `primary:`; fallback aman `ACTION_OPEN_DOCUMENT_TREE` (tidak memaksa file manager pihak ketiga membuka URI yang bukan miliknya), dan Toast jika tidak ada aplikasi.

## 6. Batas bukti

- Nilai "Prakiraan AFR" adalah perkiraan host, bukan AFR terukur; jangan dipakai untuk tuning.
- Parameter yang hanya muncul saat mesin hidup belum tentu tersedia pada semua model; ketiadaannya bukan nol.
- Pembacaan/hapus DTC telah diuji unit, belum diverifikasi pada ECU fisik dalam rebuild ini.
- Log mencatat apa yang diamati; status sesi tidak boleh disimpulkan melebihi event yang tercatat.
- Jangan menambah angka sintetis, "demo values", atau parameter placeholder untuk mengisi tabel.
