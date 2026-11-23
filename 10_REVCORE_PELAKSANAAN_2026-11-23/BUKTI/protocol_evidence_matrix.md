# RevCore — matriks bukti protokol (2026-11-22)

Kelas bukti yang dipakai:

- **CONFIRMED** — terukur/diuji langsung dan hasilnya tercatat.
- **CORROBORATED** — dua sumber independen konsisten.
- **HYPOTHESIS** — rekonstruksi masuk akal, belum diuji.
- **UNVERIFIED** — ada klaim, belum ada bukti.
- **UNSUPPORTED** — tidak ada dasar; tidak dipakai di kode.

| # | Fitur / nilai | Implementasi | Kelas bukti | Catatan |
|---|---|---|---|---|
| 1 | K-Line 10400 8N1, frame `72 len svc … chk`, checksum aditif | Tidak berubah | CONFIRMED (host) | Diverifikasi ulang oleh 394 tes termasuk codec/transceiver |
| 2 | Taksonomi kegagalan wire → session → engine | WS-A | CONFIRMED (host) | Tes pemetaan status → `WireFailureKind`; perilaku byte tidak berubah |
| 3 | Scheduler adil 4 tabel live + backoff | WS-B | CONFIRMED (host) | Tes deterministik; belum diuji pada ECU yang hanya menjawab sebagian |
| 4 | STALE per sensor 5 000 ms, nilai terakhir dipertahankan | WS-B | CONFIRMED (host) | Ambang adalah kebijakan host, bukan nilai ECU |
| 5 | Tabel ekstensi 16/20/D1 dengan rotasi dan backoff | WS-B | HYPOTHESIS | Decoder dari firmware; respons nyata belum diukur |
| 6 | AFR_est = 17,7 − 5,351·V_O2 | WS-C | HYPOTHESIS | Rumus narrowband yang diberikan; **bukan AFR terukur**, selalu berlabel estimasi host |
| 7 | O2 harus VALID/STALE, V 0,0–1,1 | WS-C | CONFIRMED (host) | Di luar itu Unavailable/Invalid |
| 8 | MODE 0x5F dan Altitude 0x79 terpisah | Tetap | UNVERIFIED | Tidak ada perubahan; tidak digabung |
| 9 | Keep-screen-on lifecycle-bound | WS-D | CONFIRMED (source) | `onStart`/`onStop`; tidak ada wakelock |
| 10 | Keyless K59J: 13 byte < 15 → SHORT_RESPONSE | WS-E | CORROBORATED | Log otopsi K59J + tes fake transport; engine tidak lagi memakai SECURITY_REJECTED |
| 11 | `PROFILE_MISMATCH` berdasarkan ambang keluarga lain | WS-E | HYPOTHESIS | Bukti terukur pada sesi, bukan kepastian model |
| 12 | Exchange Keyless bertipe (timeout/write fail) | WS-E | CONFIRMED (host) | Byte reply tidak diubah |
| 13 | Keyless RESET: 4 frame Keihin `82 82 14 08 …` / 176 frame Shindengen `91 91 09 41 …` | WS-E | CONFIRMED (host) | Konstruksi frame, checksum, timeout, dan urutan diuji dengan ECU scripted; ECU fisik belum diuji |
| 14 | Keyless RESET safety ladder (konfirmasi, prasyarat baca, cek keluarga, baca-balik, ACK≠sukses) | WS-E | CONFIRMED (host) | Tes menolak tanpa konfirmasi, menolak keluarga salah, dan melaporkan RECOVERY_REQUIRED saat link putus |
| 15 | Keyless RENEW | WS-E | CONFIRMED (host) | Firmware "Renew" menulis nol ke area yang sama dengan RESET (Keihin 0x46–0x49, Shindengen 0x00–0xAF); **bukan** pemasangan material kunci baru. `RENEW_SUPPORTED=true`; ECU fisik belum diuji |
| 16 | Remap read: unlock/seed, fast baud, blok 128 byte, fallback tabel, 3 retry, exit | WS-F | CONFIRMED (host) | Diuji pada ECU scripted; urutan byte dan tafsir balasan sesuai referensi |
| 17 | Remap write: gerbang konfirmasi, checksum image, erase, blok tulis + retry, RECOVERY saat link putus | WS-F | CONFIRMED (host) | Belum dieksekusi pada ECU nyata; success hanya diklaim setelah ACK; verifikasi terpisah |
| 18 | Shindengen seed-key (`11 12 13 14 15 15 17`, marker `EE 00/FF`) dan recovery key `91 91 0D 00 9E 8D` | WS-F | CONFIRMED (host, sintetis) | Aritmetika diuji dengan image sintetis; kunci berasal dari image pemanggil, bukan ditebak |
| 19 | Checksum image Keihin 8-bit dan Shindengen 32-bit (`D0 07`) | WS-F | CONFIRMED (host) | Vektor uji lulus; offset checksum per-profil belum diketahui dan wajib diberikan pemanggil |
| 20 | Maintenance: reset ECM, clear DTC, query flash count, reset counter Shindengen | WS-F | CONFIRMED (host) | Frame dan posisi byte balasan diuji; hasil pada kendaraan belum ada |
| 21 | Remap area reset Keihin 0x46–0x49 (4 target) | WS-F | UNVERIFIED | Frame target diketahui; geometri halaman penuh dan arti ECU atas payload belum terbukti |
| 22 | Remap area reset Shindengen 0x00–0xAF | WS-F | UNVERIFIED | Frame nol diketahui; perilaku ECU fisik belum diukur |
| 23 | Fast baud 62.500 (Keihin `7E 07 01 0E 01 90 DB`, Shindengen `7C 07 02 0E 01 00 6C`) | WS-F | UNVERIFIED | Frame literal tersedia; perilaku adapter/ECU pada baud ini belum diuji |
| 24 | Konstanta tulis 0x27/0x3D/0x7D/0x7B | WS-F | UNVERIFIED | Tetap dikarantina `@Deprecated`; jalur Remap memakai frame literal di `RemapCodec` |
| 25 | Pembacaan Keyless sukses pada kendaraan | — | UNVERIFIED | Belum ada laporan lapangan |
| 26 | Live Data akurat secara fisik | — | UNVERIFIED | Butuh alat ukur eksternal |
| 27 | Remap/Keyless write pada ECU fisik | — | UNVERIFIED | Satu-satunya bukti adalah tes host dan dekompilasi |
| 28 | H-Diag parameter/actuator | — | UNSUPPORTED (mapping) | H-Diag memakai 3 frame blok (`72 05 60 03 26`, `72 05 71 00 18`, `72 05 00 F1 98`) pada loop 100 ms, tetapi hasilnya diformat sebagai teks tanpa tabel offset/skala. Frame-nya tercatat; decoder-nya tidak dapat dibuktikan sehingga tidak dipakai |
| 31 | FlashPro probe vendor Keyless (K `82 82 10 06 00 E6`, S `91 91 07 40 4C 00 6D`), jeda 1000 ms, baca 256 alamat | WS-E | CONFIRMED (host) | Urutan probe, jeda, dan jumlah alamat diuji dengan transpor tiruan; ECU fisik belum diuji |
| 32 | FlashPro Live 7 tabel `71 xx` (300 ms) | Tetap | IMPLEMENTED BUT UNVERIFIED | Request/offset/skala eksplisit di RevCore; tidak ada respons ECU nyata dalam audit |
| 33 | Gate 10: 10 gerbang, trust record HMAC, marker instalasi, AES-256-GCM | Tetap | CONFIRMED (host) | 10/10 gerbang dipertahankan; tidak ada bypass atau pengecualian baru |
| 29 | Database 1 placeholder + 162 profil; manifest 159/2/1 | WS-H | CONFIRMED (asset) | Tes hash aset dan hitungan status |
| 30 | Gate 10 inventaris aset | WS-I | CONFIRMED (host) | Tes batas kosong/rusak/ekstra+hilang/policy kosong |
