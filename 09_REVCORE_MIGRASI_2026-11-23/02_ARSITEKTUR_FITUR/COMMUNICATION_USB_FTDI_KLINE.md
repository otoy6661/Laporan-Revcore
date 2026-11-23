# KOMUNIKASI USB, FTDI, DAN K-LINE

**Source acuan:** `core/ftdi/`, `core/usb/`, `core/protocol/`, `core/comm/`

---

## 1. Transport fisik

- Adapter yang didukung oleh kebijakan auto-connect: vendor USB
  `0x0403` (FTDI), `0x1A86`, `0x10C4`, `0x067B`
  (`core/usb/AdapterConnectionPolicy.kt`).
- Driver utama: FTDI FT232R melalui USB host Android.
- Konfigurasi source: **10400 bps, 8N1**, latency 2 ms, flow control none
  (`FtdiConstants.BAUD_KLINE_DEFAULT`, `SIO_DATA_8N1`,
  `FtdiSerialDriver`).
- Konstanta baud lain yang tersedia: 9600 (fallback baseline), 38400, 57600.
  Remap memakai 62.500 bps sebagai `FAST_BAUD_RATE`.
- Permission USB dialirkan lewat `ACTION_USB_PERMISSION`; auto-connect
  dihentikan sementara bila permission ditolak atau pengguna memilih manual.

Catatan: laporan riset lama dapat menyebut 8E1/8N1. Source current
mengonfigurasi 8N1; jangan mengubah konfigurasi tanpa bukti dan perintah Owner.

## 2. Frame K-Line

Fakta yang tercatat di `core/comm/Evidence.kt`:

| Elemen | Status | Nilai |
|---|---|---|
| Checksum | BINARY_EVIDENCE | penjumlahan semua byte sebelum checksum, dinegasi 8-bit |
| Batas panjang body | BINARY_EVIDENCE | 254 byte (`0xFE`); guard kedua 2048 byte |
| Pemilihan header request | BINARY_EVIDENCE | `0x7D` bila flag non-nol, selain itu `0x7B`, lalu `01 03` |
| Balasan minimal | BINARY_EVIDENCE | 9 byte dengan minimal 2 byte payload (konteks helper) |
| Wake pulse host | RECONSTRUCTED | BREAK LOW 70 ms, HIGH 130 ms, purge |
| Katalog request baseline | RECONSTRUCTED | frame `0x72`/`0xFE` dari baseline RevCore, bukan literal biner |

Frame yang diterima harus checksum-valid dan bukan echo lokal. `KlineTransceiver`
menyaring echo TX dalam jendela 80 ms dan menolak kandidat yang identik dengan
frame yang baru dikirim.

Status kegagalan yang dilaporkan antara lain:

- `NEGATIVE_RESPONSE` (service `0x7F`),
- `LOCAL_ECHO_ONLY`,
- `TIMEOUT`,
- `CHECKSUM_FAILED`,
- tidak ada frame valid.

## 3. Identifikasi ECU

- Query baseline pada 10400 bps; fallback 9600 bps bila tidak ada respons.
- Wake/BREAK dan query identifikasi dijalankan dalam satu lock transaksi.
- ID kanonik dipisahkan dari ID mentah; kecocokan database dan status
  kompatibilitas disimpan.
- Setelah reset ECU yang terkonfirmasi, identifikasi/Live/DTC lama dibatalkan
  dan adapter tetap terbuka.

## 4. Protocol Manager

`core/protocol/ProtocolManager.kt` adalah sumber status per protokol:

- `USB_LINK`, `ECU_IDENTIFICATION`, `LIVE_DATA`, `DTC`, `KEYLESS_ID`, `REMAP`.
- Status: `UNAVAILABLE`, `IDLE`, `READY`, `ACTIVE`, `SUSPENDED`, `FAILED`,
  `RECOVERING`.
- Lease eksklusif hanya untuk Keyless dan Remap.
- Live Data tidak boleh ACTIVE saat lease dipegang.
- Adapter ditutup/hilang → lease dicabut, status diturunkan.

Manager tidak membuka/menutup/menulis transport. Ia hanya mencatat peristiwa
yang benar-benar terjadi.

## 5. Kestabilan dan pemulihan

- `HondaProtocolEngine.requireLiveTransport()` memeriksa driver, device, dan
  `usbManager.deviceList` sebelum transaksi.
- `UsbSerialLibraryTransport` menghitung kegagalan I/O berurutan; setelah 3
  kegagalan, `isOpen = false` dan event `TRANSPORT_DEGRADED` dicatat.
- `LivePollPolicy` membatasi interval 150–1000 ms, berhenti setelah 4 kegagalan
  berturut-turut, dan memberi backoff 300–3000 ms.
- Tabel ekstensi yang gagal di-skip selama 15 detik sebelum dicoba lagi.
- Balasan yang datang setelah sesi berubah dibuang (commit `eaa9117`).

## 6. Live Data dan DTC

Detail ada di `LIVE_DATA_AFR_DTC_LOG.md`.

- Tabel yang dijawab dipertahankan; field yang tidak muat ditolak per-field.
- Respons ACK pendek tidak boleh dianggap data sensor lengkap.
- DTC Clear hanya `SUCCESS_VERIFIED` bila ECU mengonfirmasi; status MIL fisik
  tidak diklaim tanpa bit status yang sesuai.

## 7. Bukti dan batas

- `BINARY_EVIDENCE`: checksum, batas frame, header, fallback baud, perilaku
  FTDI.
- `RECONSTRUCTED`: wake timing dan katalog request baseline.
- `XREF_MAPPED`/`HYPOTHESIS`: varian H99B/KWP dan Shindengen.
- `UNRESOLVED`: beberapa frame sesi KWP dan MD 0x79.
- Semua perilaku pada adapter dan kendaraan fisik masih
  **OWNER VALIDATION PENDING**.
