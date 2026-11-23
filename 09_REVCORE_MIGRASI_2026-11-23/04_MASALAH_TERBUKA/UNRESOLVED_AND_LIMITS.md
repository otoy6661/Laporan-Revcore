# 10 — MASALAH TERBUKA DAN BATAS BUKTI

Dokumen ini mencatat hal yang **belum selesai atau tidak konsisten** pada saat backup (2026-11-23). Tidak ada item di sini yang boleh ditutup dengan nilai rekaan.

## 1. Dokumen lama yang tidak sinkron dengan source current

Source current adalah working copy di `02_CURRENT_WORKING_STATE/project`. Dokumen delivery lama **tidak diubah** (agar riwayat jujur); bila berbeda, source + laporan rebuild terbaru yang berlaku.

| # | Dokumen | Klaim lama | Keadaan source | Bukti |
|---|---|---|---|---|
| 1 | `delivery/change_summary.md:102`, `protocol_evidence_matrix.md:27`, `unresolved_items.md:35` | `RENEW_SUPPORTED=false` | `KeylessRenewPlanner.RENEW_SUPPORTED = true`; RENEW = reset tulis-nol, bukan pemasangan kunci baru | commit `329405a`, `KeylessRenewPlan.kt` |
| 2 | `delivery/unresolved_items.md` | APK resmi terakhir dibangun sebelum Keyless/Remap | Build resmi `221126` (2026-11-22 22:09) dibuat setelah pekerjaan itu | status log di `09_TESTS_AND_BUILD/build-logs/` |
| 3 | `delivery/rebuild_161126_20261122/OFFICIAL_RELEASE_221126_REPORT.md` | APK SHA-256 `62e488ac…5e04b33` | APK 221126 yang ada ber-hash `cb5892a3…ba556` (sidecar + status log + hitung ulang) | `09_TESTS_AND_BUILD/README.md` |
| 4 | `delivery/unresolved_items.md` | "462 test" | 457 test JVM pada source current; 444 saat build rilis | 55 XML `test-results-current` |
| 5 | `delivery/change_summary.md`, `live_monitor_recovery_20261122.md` | Modul `LivePollPlanner`, `LiveSampleStore`, `LiveFrameValidator`, `LiveFallbackPolicy` | Modul tersebut **tidak ada** di source current; yang ada `LivePollPolicy`, `UnifiedLiveTable`, `LiveDataPresentation`, retensi di `LiveDataProfile` | pencarian source |
| 6 | `delivery/rebuild_161126_20261122/IDENTITY_AUDIT.md` | Aset `flashpro_profiles.json` / `flashpro_maps` dipertahankan | Working tree kemudian mengganti nama aset menjadi `revcore_profiles.json` / `revcore_maps` (rename staged, isi identik) | `03_GIT_HISTORY/working-tree-status.txt` |

## 2. Protokol dan data yang belum terbukti

| Item | Keadaan | Mengapa belum tertutup |
|---|---|---|
| Skala O2 `dat20` | Nilai 3.765 V setara byte mentah 0xC0; UI menandai "di luar rentang narrowband" | Skala/offset `dat20` belum diverifikasi dengan tangkapan bus + pembanding wideband; rentang tidak dilebarkan |
| `MD 0x79` | Mode 0 vs raw belum diputuskan | Tidak ada bukti yang membedakan; sengaja tidak diubah |
| Tabel 10 (`revcore.dat10`) | Dipoll apa adanya | Provenance parameter belum jelas |
| AFR | `Prakiraan AFR` = estimasi host dari O2 narrowband | Bukan AFR terukur; butuh lambda/wideband independen |
| Keyless RESET Keihin | Frame `82 82 14 08 …` direkonstruksi | Interpretasi ECU atas separuh payload nol belum terbukti; bila ECU menulis pola literal, hasil dilaporkan PARTIAL |
| Keyless RENEW | Tulis-nol area kunci | Frame pemasangan material kunci baru tidak ada di biner; RENEW tidak boleh disebut "pasang kunci baru" |
| Remap/Flash ECU | Host protocol ADA (`WRITE_IMPLEMENTED=true`) | `WRITE_HARDWARE_VALIDATED=false`: offset checksum per profil harus diberikan pemanggil, perilaku 62 500 baud pada adapter, geometri halaman, dan recovery power-loss belum diuji |
| H-Diag | Belum ada pemetaan parameter/actuator ke request/response | Integrasi ditunda |
| `MALFORMED_FRAME` / `CHECKSUM_ERROR` | Ada di enum | Tidak pernah dipancarkan karena aturan checksum balasan vendor 0x82/0x91 belum terbukti |
| Gate 10 | Memeriksa struktur arsip/aset dan manifest UI/DEX bertanda tangan | Bukan pemeriksaan kriptografis seluruh APK di luar cakupan sembilan gate lain |

## 3. Validasi perangkat yang belum ada

Belum ada bukti dari perangkat/ECU nyata untuk rebuild ini:

- Instalasi dan startup 221126 di Poco X3 NFC (surya, Android 12/MIUI 14).
- Gate 10 first-run: exam, failure window, trust record pembuatan/penulisan, startup kedua/ketiga.
- Keystore perangkat (`[alias trust internal - tidak dipublikasikan]`, `[marker instalasi internal - tidak dipublikasikan]`) — termasuk pemulihan setelah data aplikasi dibersihkan.
- USB-OTG + adapter FTDI: auto-detect, 10400 8N1, kualitas link, reconnection.
- Live Data pada kendaraan: semua tabel, parameter tambahan saat mesin hidup, retensi STALE.
- Baca/clear DTC pada ECU.
- Keyless BACA ID, RESET, RENEW pada ECU.
- Remap baca/verifikasi/tulis pada ECU; recovery setelah listrik terputus.

Build sukses, 457 unit test, dan verifikasi APK **tidak** menggantikan pengujian ini. Keputusan penerimaan tetap milik Otoy.

## 4. Risiko keadaan kerja

- Working repo memiliki 233 entri belum-commit (185 staged, 46 unstaged, 23 entri untracked → 59 berkas). Jangan `reset --hard`/`clean`; lihat `03_GIT_HISTORY/README.md`.
- Rename aset `flashpro_*` → `revcore_*` sudah di index tetapi belum di-commit. Gate 10 di APK 221126 menginventaris nama `revcore_*`; jangan mencampur nama lama dan baru dalam satu build.
- `versionName 1.0` / `versionCode 1` tidak berubah. Bila pemilik memutuskan versi baru, `integrity_policy.json` harus diregenerasi oleh tooling rilis.
- Signing hanya boleh dilakukan pemilik lewat dialog resmi; password tidak pernah diminta/dibaca oleh Kodir.
- Repo memiliki riwayat yang di-amend (`237a946`); jangan mengandalkan commit itu, karena bisa hilang saat GC.

## 5. Pekerjaan lanjutan yang membutuhkan keputusan Otoy

1. Uji perangkat dan kirim log; bandingkan dengan matriks bukti.
2. Putuskan `MD 0x79`, provenance tabel 10, dan skala O2 setelah ada data.
3. Untuk fitur tulis pada kendaraan: siapkan ECU bench cadangan, offset checksum per profil, dan prosedur recovery power-loss yang benar-benar diuji.
4. Tinjau ulang dokumen delivery lama atau tandai superseded — jangan mengeditnya diam-diam tanpa keputusan pemilik.
5. Bila mengubah versi: regenerasi policy dan ulangi build/sign/verifikasi resmi.
