# LAPORAN BUILD RELEASE RESMI — 22 NOVEMBER 2026

**Order:** REVCORE OFFICIAL RELEASE BUILD FOR DEVICE TESTING
**Tanggal:** 2026-11-22
**Owner:** Otoy
**Executor:** Kodir
**Baseline:** Build 161126
**Branch:** `rebuild-161126`
**Working copy:** `RevCore_work_live_keyless_20261120`
**Repository:** https://github.com/otoy6661/Revcore-Project

---

## 1. Hasil

| Item | Hasil |
|---|---|
| Source yang digunakan | Working copy `RevCore_work_live_keyless_20261120`, termasuk paket koreksi absolut 22 November 2026 |
| Branch | `rebuild-161126` (HEAD `1cc9120`, tidak berubah) |
| Build Release | BERHASIL — `BUILD SUCCESSFUL in 2m 27s`, 52 task (18 executed) |
| Unit test | 444 test, 0 failure, 0 error (53 suite JVM, task `:app:testDebugUnitTest`) |
| APK berhasil dibuat | YA |
| Package name | `com.revcore.android` |
| Version name/code | `1.0` / `1` |
| APK size | 39.824.702 byte |
| APK SHA-256 | `62e488ac79f8d73531b9d8685b1a5924e6e600e4e1cfa103f0af8c3035e04b33` |
| Signing certificate SHA-256 | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| Cocok dengan sertifikat resmi | YA — sama dengan pin resmi `c649cb4f…18d098` (build 221126 sebelumnya dan 151126) |
| Gate 10 verification | Policy tertandatangani, `debug=false`, pin signer cocok; inventaris 165 aset = 165 aset APK; 3 aset terlindungi hash-nya cocok; UI/DEX manifest tertandatangani dan terikat ke policy |
| Integrity policy verification | Envelope `SHA256withRSA`, signature ada, `signerCertSha256` = cert APK, inventory dan native library cocok |
| Lokasi APK | `dist/RevCore build 221126.apk` |
| Git commit/push | TIDAK DILAKUKAN |

Checksum pendamping:
`dist/RevCore build 221126.sha256.txt` berisi hash yang sama.

---

## 2. Prosedur yang dijalankan

1. Tahap A: branch dan working copy diperiksa; `git status --short` dicatat
   sebelum build (229 entri). Keystore resmi ada (keberadaan saja; isi tidak
   dibaca), GUI signing resmi ada.
2. Unit test dijalankan ulang: 444/444 lulus.
3. Build Release dan signing dijalankan melalui
   `tools/revcore-official-release-gui.ps1`. Owner memasukkan password sendiri
   di dialog "RevCore Official Release - Signing"; password tidak pernah masuk
   ke chat, log, laporan, atau disk.
4. Pin sertifikat resmi `c649cb4f…18d098` diteruskan melalui
   `REVCORE_EXPECTED_CERT_SHA256`, sehingga build gagal bila cert tidak cocok.
5. Verifikasi independen dijalankan setelah build (bagian 3).
6. Tahap D: `git status --short` dicatat ulang (230 entri); tidak ada commit,
   push, atau perubahan branch.

---

## 3. Bukti verifikasi independen

| Pemeriksaan | Perintah | Hasil |
|---|---|---|
| Package/versi/label | `aapt2 dump badging` | `com.revcore.android` v1.0 (1), label RevCore, launchable `MainActivity` |
| Signature | `apksigner verify --verbose --print-certs` | `Verifies`; v1 false (memang dimatikan), v2 true, v3 true; 1 signer |
| Sertifikat | idem | `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta, ST=DKI Jakarta, C=ID`; SHA-256 `c649cb4f…18d098` |
| Debuggable | `aapt2 dump xmltree` | atribut `debuggable` tidak ada |
| Size/hash | `ls -l`, `sha256sum` | 39.824.702 byte; `62e488ac…4b33`; checksum file cocok |
| Integritas ZIP | `unzip -t` | No errors detected |
| Material kunci | pencarian `.p12/.jks/.keystore/[berkas kunci lokal - tidak dipublikasikan]` | tidak ada |
| Policy | decode `assets/integrity_policy.json` | `debug=false`; `expectedSignerSha256` = cert; `signerCertSha256` = cert; `algorithm=SHA256withRSA`; signature tidak kosong |
| Inventaris Gate 10 | policy vs `assets/` APK (kecuali policy, UI manifest, `dexopt/`) | 165 = 165; diff kosong; 0 kemunculan `flashpro` |
| Aset terlindungi | SHA-256 tiap aset `protected:true` | `ecm_database.json`, `revcore_profiles.json`, `revcore_maps/manifest.tsv` — ketiganya cocok |
| UI/DEX manifest | decode `assets/ui_integrity_manifest.json` | envelope `SHA256withRSA`, signer = cert resmi; `policyDigest` = SHA-256 policy yang dipaketkan (`20312235…cbfc`) |
| APK vs hasil build | `sha256sum` dua file | identik (`62e488ac…4b33`) |

Verifikasi tanda tangan policy dan manifest terhadap sertifikat resmi dilakukan
oleh `build-official-release.ps1` menggunakan sertifikat dari PKCS12 lokal;
password tidak dicetak. Verifikasi APK (v2/v3, cert, package, non-debuggable)
dilakukan ulang secara independen dengan `apksigner`/`aapt2`.

---

## 4. Koreksi build tool yang diperlukan

Saat Tahap A ditemukan referensi lama di `tools/build-official-release.ps1`
yang masih mewajibkan `flashpro_profiles.json` dan
`flashpro_maps/manifest.tsv`. Aset tersebut sudah di-rename menjadi
`revcore_profiles.json` dan `revcore_maps/manifest.tsv` oleh order pembersihan
identitas sumber. Tanpa koreksi, verifikasi build akan gagal meskipun policy
benar.

**FILE:** `tools/build-official-release.ps1`
**LOKASI:** daftar `$required` pada verifikasi policy
**GIT DIFF:**

```diff
         foreach ($required in @(
             "ecm_database.json",
-            "flashpro_profiles.json",
-            "flashpro_maps/manifest.tsv"
+            "revcore_profiles.json",
+            "revcore_maps/manifest.tsv"
         )) {
```

Verifikasi tidak dilemahkan: daftar aset wajib tetap tiga, ketiganya tetap wajib
`protected:true`, dan pemeriksaan hash/inventaris tidak berubah. Tidak ada
perubahan pada source aplikasi, Gate 10 runtime, konfigurasi signing, versi,
atau identitas.

---

## 5. Perubahan working copy akibat proses build

- Satu perubahan terlacak baru: `tools/build-official-release.ps1` (koreksi di
  bagian 4). `git status --short` 229 → 230 entri.
- Artefak baru: `dist/RevCore build 221126.apk`,
  `dist/RevCore build 221126.sha256.txt`.
- APK same-day sebelumnya (build 10:34) diarsipkan otomatis oleh prosedur resmi
  ke `dist/archive/RevCore build 221126_183735.apk` beserta `.sha256.txt`.
  Tidak ada artefak yang dihapus.
- Output Gradle berada di `app/build/` dan `build/` (tidak dilacak Git).
- HEAD tetap `1cc9120`; tidak ada commit, push, force push, perubahan branch,
  ZIP, publikasi, atau unggahan.

---

## 6. Batas status

- **Build successful** — Release resmi tertandatangani dan terverifikasi.
- **Unit tested** — 444 test lulus. Task `testReleaseUnitTest` tidak
  dikonfigurasi di proyek ini; suite JVM yang sama dijalankan lewat
  `testDebugUnitTest`.
- **Integration tested** — tidak (tidak ada perangkat pada sesi build).
- **Simulated/reconstructed** — perilaku protokol/Keyless tetap berbasis
  rekonstruksi, bukan ECU fisik.
- **Hardware not tested** — Gate 10 runtime di perangkat, USB/K-Line, ECU,
  Live Data, dan Keyless belum diuji pada perangkat.
- **Owner validation pending** — pengujian perangkat dan keputusan penerimaan
  akhir milik Owner Otoy.

---

## 7. Run ulang order yang sama — 21:23

Order build resmi yang sama diterima ulang. Source tidak berubah sejak build
18:37 (tidak ada file `app/src`, `app/build.gradle.kts`, atau `tools/` yang
lebih baru dari APK 18:37). Build dijalankan ulang melalui GUI resmi; Owner
memasukkan password kembali.

| Item | Hasil run 21:23 |
|---|---|
| APK | `dist/RevCore build 221126.apk` |
| Size | 39.824.702 byte |
| SHA-256 | `0354aff0f68ec0fc2b325d48ed2cee46e65a15666b00267365ef2642cb6e27b3` |
| Cert SHA-256 | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| Signature | v2 true, v3 true, 1 signer, not debuggable |
| Policy | signed SHA256withRSA, `debug=false`, pin cocok, 165 aset = 165 |
| Aset terlindungi | `ecm_database.json`, `revcore_profiles.json`, `revcore_maps/manifest.tsv` — hash cocok |
| Manifest UI/DEX | signed, `policyDigest` = `20312235…cbfc` |
| Checksum file | cocok dengan hash APK |
| APK hasil build Gradle | identik dengan APK ekspor |
| Git | 230 entri sebelum dan sesudah; HEAD `1cc9120`; tanpa commit/push |

APK run 18:37 (`62e488ac…4b33`) diarsipkan otomatis sebagai
`dist/archive/RevCore build 221126_212308.apk` beserta `.sha256.txt`; tidak ada
artefak yang dihapus.
