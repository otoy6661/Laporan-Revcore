# 03 — RIWAYAT GIT

Direktori ini adalah bukti Git dari **working repo** RevCore. Salinan proyek di `01_BASELINE_161126/` dan `02_CURRENT_WORKING_STATE/` sengaja dibuat **tanpa `.git`**, sehingga perbandingan baseline/current adalah perbandingan isi berkas, bukan checkout. Riwayat Git lengkap ada di sini.

## Isi direktori

| Berkas | Isi |
|---|---|
| `revcore_repository.bundle` | Bundle Git semua ref (39.8 MB), diverifikasi `git bundle verify`: "The bundle records a complete history." |
| `refs.txt` | Ref yang tersimpan: `refs/heads/master` = `e761002`, `refs/heads/rebuild-161126` = `1cc9120` |
| `branch-inventory.txt` | Dua branch lokal; `rebuild-161126` aktif |
| `commit-graph.txt` | Graf linear 9 commit (2026-11-22) |
| `commit-details.txt` | SHA penuh, author, tanggal, subjek per commit |
| `reflog.txt` | Reflog lengkap termasuk commit yang di-amend |
| `object-count.txt` | 783 objek in-pack, 2 pack, ~39.9 MB |
| `remote-inventory.txt` | Kosong — **tidak ada remote** |
| `stash-list.txt` | Kosong — **tidak ada stash** |
| `working-tree-status.txt` | `git status --porcelain=v2` working repo, 233 entri |
| `tracked-worktree-sha256.txt` | SHA-256 isi working tree untuk 632 berkas tracked |
| `untracked-sha256.txt` | SHA-256 untuk 59 berkas untracked |
| `patches/staged.patch` | Diff index terhadap HEAD (perubahan yang di-stage) |
| `patches/unstaged.patch` | Diff working tree terhadap index |
| `patches/head-to-worktree.patch` | Diff gabungan HEAD → working tree |

## Commit (urut lama → baru)

| Commit | Subjek | Peran |
|---|---|---|
| `7905a4b` | baseline: RevCore work copy as of LM-01 (2026-11-22) sebelum pemulihan total | Titik awal riwayat repo kerja |
| `e9492e9` | feat(remap): reconstructed Keihin/Shindengen read-write-verify dengan guard dan konfirmasi ketik | Pekerjaan LM-01 |
| `3c730f7` | feat(hdiag): probe halaman DTC evidence-only + guard balasan sesi basi | Pekerjaan LM-01 |
| `329405a` | feat(keyless): aktifkan RENEW sebagai reset tulis-nol firmware dengan konfirmasi ketik | Sumber kebenaran `RENEW_SUPPORTED = true` |
| `eaa9117` | fix(protocol): buang balasan yang tiba setelah sesi link berubah | Perbaikan balasan terlambat |
| `430cb6a` | feat(flashpro): rekonstruksi protokol JSON-line adapter dan sesi upload | Pekerjaan LM-01 |
| `e761002` | docs(delivery): catat build rilis resmi 221126 yang terverifikasi | `master` |
| `656bed2` | rollback: kembalikan source app ke Build 161126 | Titik rollback yang ditetapkan order |
| `1cc9120` | docs(rebuild): catat baseline Build 161126 dan arsip riset | HEAD `rebuild-161126` |

Catatan: reflog memuat `237a946` dari `commit (amend)` atas `430cb6a`. Commit itu tidak lagi dirujuk ref mana pun dan tidak dijamin ada di bundle setelah garbage collection; jangan mengandalkannya.

## Hubungan master dan rebuild-161126

- `master` (`e761002`) memuat seluruh pekerjaan LM-01 sampai laporan rilis 221126.
- `rebuild-161126` dibangun **di atas** `master`: `656bed2` mengembalikan source app ke Build 161126, `1cc9120` menambah dokumen baseline/arsip.
- Jadi riwayat versi yang lebih baru **tidak dihapus**; ia tetap dapat diakses lewat `master`.
- Rollback `656bed2` menyentuh source app, bukan dokumen delivery dan bukan aset riset.

## Keadaan working tree saat capture (2026-11-23 07:02)

Bukan commit — ini pekerjaan yang belum di-commit dan **tidak boleh dibuang**:

| Kelompok | Jumlah | Keterangan |
|---|---|---|
| Rename murni di index (`R.`) | 164 | termasuk `assets/flashpro_maps/` → `assets/revcore_maps/` dan `flashpro_profiles.json` → `revcore_profiles.json`, isi 100% identik |
| Rename + modifikasi lanjutan (`RM`) | 21 | rename di index, isi worktree diubah lagi setelahnya |
| Modifikasi belum di-stage (`.M`) | 25 | termasuk `app/build.gradle.kts`, `app/proguard-rules.pro`, `AndroidManifest.xml` |
| Entri untracked (`??`) | 23 direktori/berkas | berkembang menjadi **59 berkas** di `untracked-sha256.txt` (modul keyless/remap/livedata, tes, layar baru) |

Total entri status 233; total staged 185 (164+21); total unstaged 46 (25+21). Rename aset ke `revcore_*` adalah bagian dari pekerjaan identitas yang belum di-commit; jangan mengembalikannya ke `flashpro_*`.

## Cara memulihkan

1. Bundle: `git clone revcore_repository.bundle revcore_restore` lalu `git checkout rebuild-161126`. Ini memulihkan seluruh riwayat dan dua branch.
2. Keadaan belum-commit: gunakan salinan `02_CURRENT_WORKING_STATE/project` (sudah berisi working tree apa adanya). `patches/` adalah bukti tambahan, bukan jalur pemulihan utama.
3. Verifikasi isi dengan `tracked-worktree-sha256.txt` dan `untracked-sha256.txt`.
4. Jangan melakukan clone/apply ke dalam direktori backup; pulihkan ke lokasi kerja baru.
5. Jangan `git reset --hard`, `git clean`, atau membuang perubahan staged/untracked — semuanya mewakili pekerjaan Otoy yang belum di-commit.

## Yang tidak ada di sini

- Tidak ada remote, jadi tidak ada bukti push/publikasi dari repo ini.
- Tidak ada stash.
- Git milik proyek GUI (jika ada) tidak dibundel; yang dibundel adalah working repo `RevCore_work_live_keyless_20261120`.
- Bundle tidak memuat berkas untracked; itulah sebabnya `untracked-sha256.txt` dan salinan proyek disertakan.
