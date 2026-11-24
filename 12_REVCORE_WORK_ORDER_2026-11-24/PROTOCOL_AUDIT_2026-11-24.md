# AUDIT PROTOKOL A–E — REVCORE, 2026-11-24

Lingkup: audit protokol untuk Work Order 2026-11-24 §8.2. Sumber utama:
`delivery/protocol_evidence_matrix.md`, source `core/protocol`, `core/comm`,
`core/keyless`, `core/remap`, `data/maps`. Executor: Kodir. Pemilik/penguji final: otoy.

Kelas:
- A = sudah diimplementasikan di source dan dapat dijalankan.
- B = masih terkunci oleh mekanisme milik proyek sendiri.
- C = parsial: host sudah ada, bukti/ECU nyata belum lengkap.
- D = hanya referensi/tercatat, decoder atau pemakaian belum dapat dibuktikan.
- E = spesifikasi tidak cukup untuk implementasi tanpa mengarang.

## A — Diimplementasikan

| Fitur | Bukti | Kelas bukti |
|---|---|---|
| K-Line 10.400 8N1, frame `72 len svc … chk`, checksum aditif | codec/transceiver + test | CONFIRMED (host) |
| Taksonomi kegagalan wire → session → engine | WS-A + test pemetaan | CONFIRMED (host) |
| Scheduler Live Data 4 tabel + backoff + STALE 5.000 ms | WS-B + test deterministik | CONFIRMED (host) |
| AFR estimasi host `17,7 − 5,351·V_O2` (selalu berlabel estimasi) | WS-C + test | HYPOTHESIS (rumus), perilaku host teruji |
| Keyless ID read 256 alamat | WS-E + test fake transport | CONFIRMED (host) |
| Keyless RESET Keihin 4 frame / Shindengen 176 frame + baca-balik | WS-E + test ECU scripted | CONFIRMED (host) |
| Keyless RENEW (menulis nol ke area yang sama, bukan material kunci baru) | `KeylessRenewPlan` | CONFIRMED (host) |
| Remap read: unlock/seed, fast baud, blok 128 byte, 3 retry, exit | WS-F + test ECU scripted | CONFIRMED (host) |
| Remap write/erase host-side: konfirmasi, checksum image, erase, blok + retry, RECOVERY | WS-F | CONFIRMED (host), belum di ECU fisik |
| Shindengen seed-key dan checksum image Keihin/Shindengen | WS-F + vektor sintetis | CONFIRMED (host, sintetis) |
| Maintenance: reset ECM, clear DTC, query flash count, reset counter | WS-F | CONFIRMED (host) |
| Hapus DTC dengan verifikasi baca ulang | engine + UI | CONFIRMED (host) |

## B — Lock milik proyek

| Kandidat | Status | Tindakan |
|---|---|---|
| Flag kurasi "remap verified" pada database | Dihapus; tidak lagi menggerbangi operasi | Lock dibuka pada work order 2026-11-24 |
| RESET/RENEW Keyless diblokir label verifikasi | Tidak ada; hanya prasyarat baca + konfirmasi | Tidak ada lock untuk dibuka |
| Gerbang tulis Remap (adapter, model, vendor, ukuran, checksum, konfirmasi) | Bukan lock kurasi; prasyarat teknis + keselamatan | Dipertahankan; membuangnya akan mengarang kemampuan |
| `FlashCapabilities` (write/readback/recovery proven) | Gate bukti hardware, bukan lock fitur | Dipertahankan; host tetap dapat dipanggil lewat guard |

Kesimpulan B: tidak ditemukan lock internal yang menutup fitur yang sudah diimplementasikan.

## C — Parsial (host ada, bukti nyata belum)

| Fitur | Yang sudah ada | Yang belum |
|---|---|---|
| Tabel Live ekstensi 16/20/D1 | decoder + rotasi/backoff | respons ECU nyata |
| Akurasi fisik Live Data | permintaan/offset/skala | pengukuran alat eksternal |
| Remap write/erase | urutan host lengkap | eksekusi ECU nyata, ACK/timeout terukur, readback, recovery daya |
| Keyless RESET/RENEW | frame + ladder + baca-balik | eksekusi kendaraan |
| Fast baud 62.500 | frame literal Keihin/Shindengen | perilaku adapter/ECU pada baud ini |
| Area reset Keihin 0x46–0x49, Shindengen 0x00–0xAF | frame target | geometri halaman penuh dan arti payload |
| Maintenance reset/counter | frame + posisi byte | hasil pada kendaraan |

## D — Referensi saja

| Sumber | Temuan | Alasan tidak dipakai |
|---|---|---|
| H-Diag parameter/actuator | 3 frame blok (`72 05 60 03 26`, `72 05 71 00 18`, `72 05 00 F1 98`) loop 100 ms | hasil diformat teks tanpa tabel offset/skala; decoder tidak dapat dibuktikan |
| FlashPro probe Keyless (K/S) | urutan probe + jeda 1000 ms + 256 alamat | sudah diimplementasikan sebagai read Keyless; tidak ada perluasan baru |
| FlashPro Live `71 xx` 7 tabel | frame tercatat | tidak ada respons ECU nyata pada audit |
| Mode 0x5F dan Altitude 0x79 | tercatat terpisah | belum ada tafsir yang terbukti; tidak digabung |

## E — Spesifikasi tidak cukup

| Kasus | Keterbatasan |
|---|---|
| Kontainer MAP FlashPro | tidak memuat ECU ID, signature, atau checksum yang mengikat image ke ECU; kompatibilitas fisik tidak dapat dibuktikan dari file |
| Konstanta tulis 0x27/0x3D/0x7D/0x7B | tetap `@Deprecated`; pemakaian akan mengarang protokol |
| Kode DTC 0x3000 | tidak ada deskripsi terverifikasi; hanya pasangan mentah 0x30/0x00 |
| Geometri EEPROM/keyless di luar area yang terbaca | tidak ada sumber; tidak dibuat |

## Catatan keputusan

- Tidak ada frame, checksum, seed-key, geometri, atau timing baru yang dikarang pada
  work order ini.
- Pembukaan lock hanya menghapus gerbang status kurasi; prasyarat teknis dan
  konfirmasi eksplisit tetap berlaku.
- Status FIXED berarti source/test terpasang; validasi perangkat/ECU fisik adalah
  keputusan pemilik.
