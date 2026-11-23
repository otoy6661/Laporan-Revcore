# 08 — ARSITEKTUR DAN FITUR

Indeks dokumen:

1. `APP_ARCHITECTURE.md` — lapisan aplikasi, alur startup, data, dan build.
2. `COMMUNICATION_USB_FTDI_KLINE.md` — transport, protokol, sesi, kegagalan.
3. `VEHICLE_DATABASE_AND_IDENTITY.md` — database ECM, identitas, aset.
4. `MAP_TUNING_AND_REMAP.md` — MAP, editor, BIN, remap, batas tulis.
5. `KEYLESS_ID.md` — Keyless read/reset/renew dan bukti firmware.
6. `LIVE_DATA_AFR_DTC_LOG.md` — polling, validasi, retensi, AFR, DTC, log.

Dokumen-dokumen ini menjelaskan **source current** di
`02_CURRENT_WORKING_STATE/project`. Laporan `delivery/change_summary.md` dan
`delivery/live_monitor_recovery_20261122.md` berasal dari pekerjaan LM-01
sebelum rollback dan memuat beberapa modul yang **tidak ada** di source current
(misalnya `LivePollPlanner`, `LiveSampleStore`, `LiveFrameValidator`,
`LiveFallbackPolicy`). Perbedaan itu dicatat di
`10_KNOWN_ISSUES/UNRESOLVED_AND_LIMITS.md`; jangan menganggap laporan lama
sebagai deskripsi source saat ini.

**Prinsip bukti:** setiap elemen komunikasi memiliki status
`BINARY_EVIDENCE`, `XREF_MAPPED`, `RECONSTRUCTED`, `HYPOTHESIS`, atau
`UNRESOLVED` (`core/comm/Evidence.kt`). Status tidak boleh dinaikkan tanpa bukti.
