# EXCLUSIONS AND LIMITS

Dokumen ini menyatakan apa yang sengaja **tidak** dimasukkan, apa yang dikecualikan dari salinan, dan batas yang membuat backup ini tidak boleh disebut "terverifikasi sepenuhnya" di luar cakupannya.

## 1. Dikecualikan dari salinan proyek

Log robocopy `01_BASELINE_161126/copy-project.log` dan `02_CURRENT_WORKING_STATE/copy-project.log` mencatat pengecualian yang sama: `build`, `.gradle`, `.kotlin`, `.git`, `dist`.

| Item | Alasan | Di mana buktinya |
|---|---|---|
| `build/`, `.gradle/`, `.kotlin/` | Output/cache yang dapat diregenerasi; menyalinnya akan mencampur artefak basi dengan source | log robocopy |
| `.git/` | Bukan isi proyek; seluruh riwayat dibundel terpisah | `03_GIT_HISTORY/revcore_repository.bundle` + verifikasi clone |
| `dist/` | Disalin terpisah, tidak dicampur ke dalam pohon source | `01_BASELINE_161126/dist/`, `02_CURRENT_WORKING_STATE/dist/` |

`tools/__pycache__` tetap ada karena berada di pohon proyek; isinya bukan kode sumber yang aktif.

## 2. Tidak ada rahasia di dalam paket migrasi

- Tidak ada `*.key`, `*.p12`, `*.jks`, `debug.keystore`, atau file keystore lain di seluruh `REVCORE_AI_MIGRATION_BACKUP` (pencarian nama file; hasil kosong).
- Key pack dan keystore asli hanya ada di `[folder kunci lokal - tidak dipublikasikan]` dan salinan privat di `[lokasi cadangan kunci privat - tidak dipublikasikan]`, keduanya **di luar** paket ini dan tidak dipublikasikan.
- Tidak ada password, isi key pack, atau materi rahasia yang dibaca/ditulis di dokumen, log, atau manifest. Hash yang dicantumkan adalah hash kontainer, bukan isi.
- `H-Diag/login.ini` dan kredensial serupa di dalam `Bahan Otopsi.zip` tidak dibuka, tidak diekstrak, dan tidak disalin keluar.
- Dua salinan `debug.keystore` yang sebelumnya ikut tersapu ke salinan proyek telah dihapus dari paket (bukan dari lokasi aslinya); debug keystore bukan material signing resmi.

## 3. Tidak menyentuh sumber/arsip asli

- `C:\Users\Lord Otoy\Desktop\Bahan Otopsi.zip` tidak diubah, tidak di-rename, tidak di-rezip. Hash ulang 2026-11-23 tetap `b9db1554…ae1b9`.
- `work/revcore` (baseline) tidak ditimpa oleh working copy.
- Tidak ada Git commit, push, branch change, release, atau publikasi yang dilakukan untuk membuat backup ini.
- Dokumen delivery lama yang tidak konsisten **tidak diedit**; selisih dicatat di `10_KNOWN_ISSUES/UNRESOLVED_AND_LIMITS.md`.

## 4. Batas cakupan verifikasi

| Hal | Status |
|---|---|
| Setiap file dalam paket vs `SHA256SUMS.txt` | Diverifikasi `sha256sum -c`: 3628/3628 OK |
| Baseline project vs salinannya | Dibandingkan hash per file; diff kosong (`11_MIGRATION_MANIFEST/baseline_sha256.diff`, `baseline_copy_filelist.diff`) |
| Current project vs salinannya | Dibandingkan hash per file; diff kosong (`current_sha256.diff`, `current_copy_filelist.diff`) |
| Bundle Git | `git bundle verify` OK; clone uji ke direktori sementara berhasil, 9 commit, `master` + `rebuild-161126` |
| APK 161126/221126/201126 | Hash ulang = sidecar; v2+v3 valid; cert pin cocok; tidak debuggable |
| Restore isi keystore | **Tidak diuji** — memerlukan password; dilarang |
| Kesetaraan `.jks` vs `.p12` | **Tidak diuji** |
| Instalasi APK / Gate 10 di perangkat | **Belum dilakukan** — wewenang Otoy |
| Uji ECU fisik (Live/Keyless/Remap) | **Belum dilakukan** |
| UI di browser/emulator | Tidak berlaku (aplikasi Android native) dan tidak dijalankan |

## 5. Batas snapshot

- `02_CURRENT_WORKING_STATE/project` adalah snapshot working tree pada 2026-11-23 07:02, termasuk 233 entri belum-commit. Bila proyek asli diubah setelah itu, snapshot ini tidak lagi "current"; `03_GIT_HISTORY/tracked-worktree-sha256.txt` dan `untracked-sha256.txt` adalah pembanding isi saat capture.
- Bundle Git tidak memuat berkas untracked; itulah sebabnya salinan proyek dan hash untracked disertakan.
- Riwayat yang di-amend (`237a946`) tidak dirujuk ref dan tidak dijamin ada setelah GC.
- Build `201126` disertakan karena berada di `dist` saat capture; ia bukan baseline dan bukan rilis yang diserahkan.
- `OFFICIAL_RELEASE_221126_REPORT.md` memuat hash APK yang salah; nilai yang berlaku adalah sidecar/status log/hitung ulang (`cb5892a3…ba556`).

## 6. Privasi

Paket ini memuat source, firmware/APK pihak ketiga di dalam `Bahan Otopsi.zip`, nama pemilik, dan path lokal. Perlakukan seluruh paket sebagai **privat**. Jangan unggah ke layanan AI publik, repository publik, atau cloud tanpa perintah eksplisit Otoy.
