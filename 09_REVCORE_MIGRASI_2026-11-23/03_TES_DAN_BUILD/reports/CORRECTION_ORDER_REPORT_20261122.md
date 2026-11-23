# LAPORAN PELAKSANAAN — KOREKSI RELEASE REVCORE

**Order:** WORK ORDER — KOREKSI RELEASE REVCORE (22 November 2026)
**Owner:** Otoy
**Executor:** Kodir
**Baseline sumber:** working copy `RevCore_work_live_keyless_20261120`, branch `rebuild-161126`, HEAD `1cc9120`
**Baseline UI sebelum koreksi ini:** snapshot file-history 22 Nov 2026 17:28/18:22 (sebelum suntingan order ini); diff lengkap ada di `diffs_2130/`
**Tanggal laporan:** 2026-11-22
**Status:** implementasi + pengujian software selesai; APK Release resmi disiapkan melalui mekanisme signing Owner; penerimaan akhir milik Owner.

---

## 0. Ringkasan kepatuhan

| Koreksi | Status | Bukti utama |
|---|---|---|
| 1 — Loading screen dipertahankan | TIDAK DIUBAH | Tidak ada perubahan pada `StartupState`, `StartupTiming`, `StartupHoldScreen`, `LoadingScreen`; `StartupTimingTest` 3/3 lulus |
| 2 — Live Data & Prakiraan AFR | DIIMPLEMENTASIKAN | `LiveDataPresentation.kt` (baru), `LiveDataScreen.kt`, `UnifiedLiveTable.kt`; 12 test baru/berubah |
| 3 — Urutan & status Keyless ID | DIIMPLEMENTASIKAN | `KeylessScreen.kt` ditulis ulang, `KeylessUiState.kt`, `MainViewModel.kt`; `KeylessUiStateTest` 7/7 |
| 4 — Tampilan/alur Tuning | DIIMPLEMENTASIKAN | `TuningScreen.kt`: tiga bagian bernomor, MAP sebelum BIN/remap, Advanced/ROM tools ditampilkan |
| APK Release resmi | DIBUAT & DIVERIFIKASI | `dist/RevCore build 221126.apk`; signing resmi, Gate 10 cocok 165/165 aset (bagian 8) |

Batas yang dijaga:

- Tidak ada perubahan `versionName`, `versionCode`, `applicationId`, identitas Owner, atau konfigurasi signing.
- Tidak ada commit, push, perubahan branch, ZIP distribusi, publikasi, atau unggahan.
- Gate 10 tidak dilemahkan/dinonaktifkan; tidak ada aset, policy, atau pin sertifikat yang diubah.
- Tidak ada protokol, alamat, checksum, rumus AFR, key material, atau hasil uji ECU fisik yang dikarang.
- APK Debug tidak diserahkan sebagai pengganti APK Release.

---

## 1. KOREKSI 1 — Loading screen: regresi saja

**Tidak ada perubahan source startup pada order ini.**

Yang diverifikasi ulang:

- Alur tetap: Gate 10 / trust record → tahan 2 detik (`StartupTiming.HOLD_BEFORE_LOADING_MS = 2000`) → animasi loading → menu.
- `loadingAllowed()` tidak berubah; trust record tetap boleh melewati konfirmasi Gate 10 tetapi tidak melewati animasi loading.
- Tidak ada penggantian animasi dengan progress bar statis atau tampilan debug.
- Kebijakan Gate 10 tidak diubah.

**TEST:** `:app:testDebugUnitTest` → `StartupTimingTest` 3 test, 0 gagal. Full suite 457 test, 0 gagal.

**GIT DIFF:** tidak ada untuk file startup.

---

## 2. KOREKSI 2 — Live Data & Prakiraan AFR

Prinsip yang dipakai: **hanya presentasi**. Mekanisme pembacaan, decoding, polling, tabel Honda, dan `LiveTelemetry` tidak diganti. Tidak ada Role Table baru; `sourceTable`/`sourceOffset` tetap hanya provenance teknis di model baris.

### 2.1 `UnifiedLiveTable.kt` — hanya parameter yang benar-benar diterima

- **FILE:** `app/src/main/java/com/revcore/android/core/comm/UnifiedLiveTable.kt`
- **LOKASI:** `object UnifiedLiveTable` → `fun build(telemetry, now)`
- **ACTUAL OLD CODE:**
```kotlin
val seen = LinkedHashMap<LiveSensor, UnifiedLiveRow>()
for (profile in HondaLiveDataProfiles.tables) {
    for (field in profile.fields) {
        if (field.sensor in seen) continue
        seen[field.sensor] = row(telemetry, field.sensor, profile.tableId, field.offset, field.unit, now)
    }
}
...
for (sensor in telemetry.parameterStates.keys) {
    if (sensor in seen) continue
    seen[sensor] = row(telemetry, sensor, "-", -1, unitOf(sensor), now)
}
```
- **REPLACEMENT CODE:**
```kotlin
val present = telemetry.parameterStates.keys + telemetry.readings.keys
val seen = LinkedHashMap<LiveSensor, UnifiedLiveRow>()
for (profile in HondaLiveDataProfiles.tables) {
    for (field in profile.fields) {
        if (field.sensor in seen) continue
        if (field.sensor !in present) continue
        seen[field.sensor] = row(telemetry, field.sensor, profile.tableId, field.offset, field.unit, now)
    }
}
for (sensor in present) {
    if (sensor in seen) continue
    seen[sensor] = row(telemetry, sensor, "-", -1, unitOf(sensor), now)
}
```
- **REASON:** katalog tidak boleh menjadi daftar isian yang memunculkan sensor yang belum pernah dijawab/diparse ECU.
- **IMPACT:** tabel tidak lagi berisi baris sensor yang belum pernah diterima; sensor yang diterima tetapi tidak ada di katalog tetap tampil dengan sumber `"-"`.
- **TEST:** `UnifiedLiveTableTest` 8 test (termasuk `catalogSensorThatWasNeverReceivedIsNotListed`).
- **GIT DIFF:** `diffs_2130/UnifiedLiveTable.diff` (+6/−1).

### 2.2 `LiveDataPresentation.kt` — pane dan AFR (baru)

- **FILE:** `app/src/main/java/com/revcore/android/ui/livedata/LiveDataPresentation.kt` (baru)
- **LOKASI:** `enum LiveDataPane`, `object LiveDataPresentation`
- **ACTUAL OLD CODE:** tidak ada; keputusan tampil sebelumnya dilakukan langsung di layar (AFR selalu dirender).
- **REPLACEMENT CODE (inti):**
```kotlin
enum class LiveDataPane { CONNECT_VEHICLE, WAITING_FOR_DATA, LIVE }

fun pane(adapterConnected: Boolean, hasReceivedData: Boolean): LiveDataPane = when {
    !adapterConnected -> LiveDataPane.CONNECT_VEHICLE
    !hasReceivedData -> LiveDataPane.WAITING_FOR_DATA
    else -> LiveDataPane.LIVE
}

fun hasReceivedData(state: LiveDataState, isValid: Boolean, hasRows: Boolean): Boolean =
    isValid || hasRows ||
        state == LiveDataState.ACTIVE || state == LiveDataState.STALE

fun showsAfr(o2State: LiveParameterState?): Boolean = o2State == LiveParameterState.VALID
```
- **REASON:** memisahkan tiga kondisi (belum terhubung / menunggu data pertama / sudah pernah menerima data) dan memaksa AFR hanya tampil untuk O2 `VALID`.
- **IMPACT:** setelah satu frame diterima, tabel tetap tampil walau polling berikutnya gagal; baris menjadi `STALE` dengan nilai asli dan usia, bukan angka baru. Saat koneksi baru dibuka, `LiveDataState` kembali `UNAVAILABLE`, jadi tabel tidak menampilkan sisa sesi sebelumnya.
- **TEST:** `LiveDataPresentationTest` 5 test.
- **GIT DIFF:** `diffs_2130/LiveDataPresentation.diff` (file baru).

### 2.3 `LiveDataScreen.kt` — tampilan sebelum/sesudah data valid

- **FILE:** `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt`
- **LOKASI:** `@Composable fun LiveDataScreen`, `UnifiedLiveRowView`, helper baru `LiveDataMessageCard`
- **ACTUAL OLD CODE (potongan yang diubah):**
```kotlin
val hasValidData = telemetry.isDataValid
// ...layar langsung merender subheader, hero, tabel, sensor, lalu:
// "Prakiraan AFR" is always present: ...
AfrEstimateCard(estimate = afrEstimate, telemetry = telemetry, isId = isId)
...
if (quickItems.isEmpty()) {
    Text(if (isId) "Belum ada parameter yang dapat ditampilkan dari respons ECU." else "...")
}
...
if (sensorCards.isEmpty()) {
    Text(if (isId) "Tidak ada parameter sensor yang dapat ditampilkan..." else "...")
}
...
Text(
    text = if (row.sourceTable == "-") ... else "dat${row.sourceTable}" +
        if (row.sourceOffset >= 0) " · byte ${row.sourceOffset}" else "" ...
)
```
- **REPLACEMENT CODE (inti):**
```kotlin
val hasReceivedInSession = LiveDataPresentation.hasReceivedData(
    liveDataState, hasValidData, telemetry.parameterStates.isNotEmpty()
)
val pane = LiveDataPresentation.pane(isConnected, hasReceivedInSession)
val o2State = telemetry.stateOf(LiveSensor.O2)
val showAfr = pane == LiveDataPane.LIVE && LiveDataPresentation.showsAfr(o2State)

if (pane == LiveDataPane.CONNECT_VEHICLE) {
    LiveDataMessageCard(title = "KENDARAAN BELUM TERHUBUNG", message = "...", ...)
    return@Column
}
...
if (pane == LiveDataPane.WAITING_FOR_DATA) {
    LiveDataMessageCard(title = "MENUNGGU DATA KENDARAAN", message = "...", ...)
    return@Column
}
...
if (quickItems.isNotEmpty()) { ... }        // tidak ada teks kosong internal
if (sensorCards.isNotEmpty()) { ... }       // judul + kartu hanya bila ada isi
if (showAfr) { AfrEstimateCard(...) }       // AFR hanya saat O2 VALID
...
if (row.state == LiveParameterState.STALE && row.ageMs != null) {
    Text("data ${row.ageMs / 1000}s lalu")
}
```
- **REASON:** memenuhi A–E: pesan sederhana sebelum terhubung, menunggu respons pertama tanpa tabel kosong, hanya parameter yang benar-benar diterima, AFR tidak tampil sebelum O2 valid, dan tidak ada pesan diagnostik internal di layar pengguna.
- **IMPACT:** sebelum terhubung hanya satu kartu instruksi; setelah terhubung tetapi belum ada data hanya kartu menunggu; setelah data valid, tabel + kartu sensor + AFR (bila O2 valid) muncul otomatis. Provenance `dat… / byte …` tidak lagi ditampilkan sebagai noise; tetap tersimpan di model baris untuk ketertelusuran.
- **TEST:** `LiveDataPresentationTest`; `UnifiedLiveTableTest`; kompilasi `LiveDataScreen`.
- **GIT DIFF:** `diffs_2130/LiveDataScreen.diff`.

### 2.4 `AfrEstimator.kt` — verifikasi rumus (tidak diubah)

- **FILE:** `app/src/main/java/com/revcore/android/core/comm/AfrEstimator.kt`
- **TEMUAN:** rumus sudah sesuai riset: `AFR = 17.700001 − 5.351 × V_O2`, dengan `O2_COUNTS_PER_VOLT = 50.993698`, rentang 0.0–1.1 V, label `"Prakiraan AFR"`.
- **KEPUTUSAN:** tidak ada perubahan rumus. Yang diubah hanya kapan kartu boleh tampil (2.3).
- **TEST:** `AfrEstimatorTest` (rumus, O2 hilang, state non-VALID, di luar rentang, non-numerik, label).

---

## 3. KOREKSI 3 — Keyless ID: urutan, status, dan pengaktifan

### 3.1 `KeylessUiState.kt` — status tiga-nilai dan hasil per aksi

- **FILE:** `app/src/main/java/com/revcore/android/ui/keyless/KeylessUiState.kt`
- **LOKASI:** `KeylessStepStatus`, `KeylessUiState`
- **ACTUAL OLD CODE:**
```kotlin
data class KeylessUiState(
    val stage: KeylessStage = KeylessStage.IDLE,
    ...
    val resetRunning: Boolean = false,
    val resetProgress: Float = 0f,
    val resetMessage: String = "",
    val resetReport: KeylessResetReport? = null
) {
    val running: Boolean get() = stage.running
    val busy: Boolean get() = running || resetRunning
    val matched: KeylessProfileResolver.Resolution.Matched?
        get() = resolution as? KeylessProfileResolver.Resolution.Matched
}
```
- **REPLACEMENT CODE (inti):**
```kotlin
enum class KeylessStepStatus { WAITING, NOT_AVAILABLE, FAILED, SUCCESS }

val readResultValid: Boolean = false,
val readProfileId: String? = null,
val resetAction: KeylessMaintenanceAction? = null,
val resetReportByAction: Map<KeylessMaintenanceAction, KeylessResetReport> = emptyMap()

val readStatus: KeylessStepStatus get() = when {
    running -> WAITING
    report?.success == true && readResultValid -> SUCCESS
    stage == KeylessStage.FAILED || stage == KeylessStage.CANCELLED -> FAILED
    else -> WAITING
}

fun maintenanceStatus(action: KeylessMaintenanceAction): KeylessStepStatus {
    val result = resetReportByAction[action] ?: return NOT_AVAILABLE
    return if (result.outcome == KeylessProcessOutcome.SUCCESS) SUCCESS else FAILED
}

fun canRunMaintenance(adapterConnected: Boolean): Boolean =
    adapterConnected && !busy && matched != null && readResultValid && report?.success == true
```
- **REASON:** BACA memakai Menunggu/Gagal/Berhasil; RESET dan RENEW memakai Belum tersedia/Gagal/Berhasil dan tidak saling menimpa; BERHASIL hanya berasal dari outcome yang sudah diverifikasi read-back, bukan ACK.
- **IMPACT:** status RESET dan RENEW terpisah; tombol hanya aktif setelah BACA berhasil dan hasilnya masih valid untuk profil saat ini.
- **TEST:** `KeylessUiStateTest` 7 test (termasuk ACK-only `UNKNOWN` bukan SUCCESS, RESET SUCCESS vs RENEW PARTIAL, busy).
- **GIT DIFF:** `diffs_2130/KeylessUiState.diff`.

### 3.2 `MainViewModel.kt` — validitas hasil baca dan pembatalan

- **FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`
- **LOKASI:** `refreshKeylessProfile`, `invalidateKeylessRead`, `startKeylessRead`, `startKeylessReset`
- **ACTUAL OLD CODE (potongan yang diubah):**
```kotlin
if (state.report?.success != true) {
    val reason = text(
        "Renew/Reset hanya dijalankan setelah Keyless ID berhasil dibaca dan diperiksa. " +
            "Lakukan pembacaan ID terlebih dahulu.", "..."
    )
    ...
}
...
_keylessState.update { it.copy(resetRunning = true, resetProgress = 0f,
    resetReport = null, resetMessage = ...) }
...
_keylessState.update { it.copy(resetRunning = false, resetProgress = 1f,
    resetReport = report, resetMessage = keylessResetMessage(report)) }
```
- **REPLACEMENT CODE (inti):**
```kotlin
// hasil baca hanya sah untuk profil asalnya
val nextProfileId = (next as? Resolution.Matched)?.profile?.id
val before = _keylessState.value
val profileChanged = before.readResultValid && before.readProfileId != null &&
    before.readProfileId != nextProfileId
val identityLost = next !is Resolution.Matched && (before.readResultValid || before.report != null)
if (!before.running && (profileChanged || identityLost)) { /* reset report + validity */ }

fun invalidateKeylessRead() {   // dipanggil layar saat adapter terputus
    if (state.running || state.busy) return
    _keylessState.update {
        it.copy(stage = IDLE, report = null, readResultValid = false, readProfileId = null,
            resetReport = null, resetReportByAction = emptyMap(), resetAction = null, ...)
    }
    CommunicationSniffer.logEvent(..., status = "KEYLESS_ID_READ_INVALIDATED", ...)
}

// mulai baca baru: hapus hasil lama
readResultValid = false, readProfileId = null, resetReport = null,
resetReportByAction = emptyMap(), resetAction = null
// selesai baca
readResultValid = report.success
readProfileId = if (report.success) report.model.id else null
// guard RESET/RENEW
if (state.report?.success != true || !state.readResultValid) { ...refuse... }
// mulai/selesai aksi
resetAction = action
resetReportByAction = it.resetReportByAction + (action to report)
resetAction = action
```
- **REASON:** hasil baca tidak boleh bertahan setelah adapter putus atau identitas berubah; RESET dan RENEW harus dievaluasi ulang dan menyimpan hasil masing-masing.
- **IMPACT:** popup/ID hilang saat putus; tombol kembali nonaktif; guard ViewModel menjadi lapisan kedua setelah UI.
- **TEST:** `KeylessUiStateTest`; seluruh suite Keyless lama tetap lulus.
- **GIT DIFF:** `diffs_2130/MainViewModel.diff` (+79/−6).

### 3.3 `KeylessScreen.kt` — urutan 1–5, popup, dan transparansi

- **FILE:** `app/src/main/java/com/revcore/android/ui/screens/KeylessScreen.kt`
- **LOKASI:** seluruh layar; helper `VehicleSection`, `EcmSection`, `ReadSection`, `MaintenanceSection`, `StatusRow`, dialog konfirmasi
- **ACTUAL OLD CODE (struktur sebelum order):**
```kotlin
Text("KEYLESS ID READER", ...)
// kartu adapter/ECU/kontak
// satu kartu "KENDARAAN TERIDENTIFIKASI" + profil otomatis + alamat EEPROM
Row { Button("BACA ID", enabled = isConnected && !state.running && selectedModel != null) ... }
if (report != null) ResultCard(...)
// kartu RESET dan RENEW dengan enabled = isConnected && !busy && selectedModel != null &&
//     state.report?.success == true
// kartu "BATASAN BUKTI"
// tidak ada popup hasil baca dan tidak ada pemisahan status per aksi
```
- **REPLACEMENT CODE (struktur):**
```kotlin
Text("KEYLESS ID", ...)
KeylessCard(keyless_vehicle_card)  { SectionHeader("1", "KENDARAAN TERIDENTIFIKASI") ... }
KeylessCard(keyless_ecm_card)      { SectionHeader("2", "ECM") ... }
KeylessCard(keyless_read_card)     { SectionHeader("3", "BACA KEY ID")
    StatusRow(state.readStatus, maintenance = false)
    Button(btn_keyless_read, enabled = isConnected && !state.busy && selectedModelAvailable)
    // sukses: ID 30sp + Salin ID + Salin laporan + badge bukti + detail teknis
    // gagal: teks kegagalan
}
MaintenanceSection(RESET, "4. RESET KEY ID") // Belum tersedia/Gagal/Berhasil
MaintenanceSection(RENEW, "5. RENEW KEY ID") // Belum tersedia/Gagal/Berhasil
EvidenceLimitsCard()

LaunchedEffect(isConnected) { if (!isConnected) viewModel.invalidateKeylessRead() }
LaunchedEffect(state.report?.keyId, state.readResultValid) { /* buka/tutup popup */ }
AlertDialog(keyless_key_id_popup) { /* Key ID + Salin ID */ }
```
- **REASON:** memenuhi urutan wajib Owner, status per langkah, popup hanya untuk ID yang benar-benar berhasil dibaca, dan RESET/RENEW hanya aktif setelah hasil baca masih valid.
- **IMPACT:** identitas kendaraan hanya dari database yang cocok; tipe ECM tidak ditampilkan sebagai KEIHIN/SHINDENGEN bila belum teridentifikasi; ID tidak pernah ditampilkan dari baca gagal; RESET dan RENEW tetap dua operasi terpisah; teks menyatakan keduanya menulis nol ke area yang sama dan RENEW bukan pemasangan kunci baru.
- **TEST:** kompilasi; `KeylessUiStateTest`; suite Keyless lama (frame codec, decoder, resolver, read/reset engine) tetap lulus. Tidak ada uji ECU fisik.
- **GIT DIFF:** `diffs_2130/KeylessScreen.diff` (tulis ulang; +784/−597).

Catatan bukti: protokol, urutan frame, alamat, panjang, checksum, timeout, delay, dan read-back tidak diubah. `PROTOCOL_ANALYZED` dan `FIELD_TEST_PENDING` tetap dibedakan; tidak ada status `FIELD_VERIFIED` yang dibuat.

---

## 4. KOREKSI 4 — Tuning: struktur dan alur

- **FILE:** `app/src/main/java/com/revcore/android/ui/screens/TuningScreen.kt`
- **LOKASI:** `fun TuningScreen`; helper baru `TuningSectionHeader`; pemanggilan `AdvancedSection`
- **ACTUAL OLD CODE (urutan sebelum order):**
```kotlin
Text("TUNING", ...)
EcmPanel(...)
RemapWorkflowCard(...)
RemapExecutionCard(...)
if (selectedModel?.isVerifiedRemap != true) { /* kartu dukungan remap */ }
// kemudian baru MAP: MapSelectionCard / MapWorkspaceHeader + editor + operasi
// AdvancedSection tidak dipanggil; log muncul di dalam AdvancedSection
```
- **REPLACEMENT CODE (urutan):**
```kotlin
Column { Text("TUNING", ...); Text("Kendaraan → MAP → BIN dan proses remap. ...") }
TuningSectionHeader("1", "KENDARAAN & ECM", "...")
EcmPanel(...)
TuningSectionHeader("2", "MAP", "...")
if (map == null) MapSelectionCard(...) else { MapWorkspaceHeader + Category/Section + editor
    + DiffCard + TuningOperationsCard(backup/export/checksum/write) }
TuningSectionHeader("3", "BIN & PROSES REMAP", "...")
RemapWorkflowCard(...)
RemapExecutionCard(...)
if (selectedModel?.isVerifiedRemap != true) { /* "Write BIN tetap terkunci sampai
    protokol tulis dan verifikasi perangkat keras terbukti." */ }
AdvancedSection(...)   // alat ROM/BIN lama kini benar-benar ditampilkan
OutlinedButton("Tampilkan LOG") + TuningLogPanel(...)
```
- **REASON:** menyusun hierarki sesuai pola alur FlashPro (kendaraan/ECM → MAP → BIN/remap), tanpa mengganti identitas RevCore dan tanpa menghapus fungsi.
- **IMPACT:** MAP dipilih sebelum operasi BIN/remap; panel MAP menampilkan metadata/profil dan editor; operasi backup/ekspor/checksum/write tetap ada; alat advanced/ROM yang sebelumnya terdefinisi tetapi tidak dipanggil kini tersedia dalam satu bagian yang dapat dibuka; log tidak lagi ganda.
- **TEST:** kompilasi; seluruh suite Tuning/Map/Remap tetap lulus (457 total). TestTag lama dipertahankan (`tuning_ecm_panel`, `tuning_map_select`, `btn_backup_bin`, `btn_export_map`, `btn_verify_checksum`, `btn_write_bin`, `tuning_operations`, `tuning_remap_execution`, `tuning_remap_workflow`, `tuning_advanced`, `tuning_flash_gate`, dll.).
- **GIT DIFF:** `diffs_2130/TuningScreen.diff` (+163/−90).

Batas klaim: `RemapWorkflow` tetap menetapkan `WRITE_HARDWARE_VALIDATED = false`; tidak ada klaim bahwa Write BIN sudah terbukti. Pemilihan dan ekspor MAP tetap tersedia.

---

## 5. File baru / berubah / dihapus / di-rename (order ini)

**Baru:**
- `app/src/main/java/com/revcore/android/ui/livedata/LiveDataPresentation.kt`
- `app/src/test/java/com/revcore/android/ui/keyless/KeylessUiStateTest.kt`
- `app/src/test/java/com/revcore/android/ui/livedata/LiveDataPresentationTest.kt`
- `delivery/rebuild_161126_20261122/diffs_2130/*.diff` (bukti diff; bukan source aplikasi)

**Berubah:**
- `app/src/main/java/com/revcore/android/core/comm/UnifiedLiveTable.kt`
- `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt`
- `app/src/main/java/com/revcore/android/ui/keyless/KeylessUiState.kt`
- `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`
- `app/src/main/java/com/revcore/android/ui/screens/KeylessScreen.kt`
- `app/src/main/java/com/revcore/android/ui/screens/TuningScreen.kt`
- `app/src/test/java/com/revcore/android/core/comm/UnifiedLiveTableTest.kt`

**Dihapus / di-rename:** tidak ada pada order ini.

Catatan: `KeylessScreen.kt`, `KeylessUiState.kt`, `UnifiedLiveTable.kt`, dan `LiveDataPresentation.kt` masih untracked terhadap Git (warisan pekerjaan sebelumnya di working copy). Karena itu bukti diff memakai snapshot file-history sebelum suntingan, bukan `git diff` HEAD.

---

## 6. Kompilasi dan pengujian

Lingkungan (tanpa perubahan konfigurasi proyek):

```
JAVA_HOME="C:/Program Files/Android/Android Studio/jbr"
gradle.bat --offline -Prevcore.publishDist=false :app:testDebugUnitTest
```

| Pemeriksaan | Hasil |
|---|---|
| `:app:compileDebugKotlin` | BUILD SUCCESSFUL |
| `:app:testDebugUnitTest` | BUILD SUCCESSFUL |
| Suite | 55 |
| Test | **457** |
| Gagal / error / dilewati | 0 / 0 / 0 |
| `KeylessUiStateTest` | 7/7 |
| `LiveDataPresentationTest` | 5/5 |
| `UnifiedLiveTableTest` | 8/8 |
| `StartupTimingTest` | 3/3 (KOREKSI 1) |
| `IntegrityGateTest` | 31/31 |
| `GateStagingTest` | 13/13 |

Baseline sebelum order ini: 444 test, 53 suite, 0 gagal. Penambahan 13 test berasal dari status Keyless, presentasi Live Data, dan filter tabel.

Yang **belum** diuji: perangkat fisik, USB-OTG/K-Line, ECU, respons O2 nyata, pembacaan/reset Keyless pada kendaraan, dan perilaku Gate 10 saat runtime di perangkat. Itu tetap tugas pengujian Owner.

---

## 7. Gate 10 dan integritas

- Tidak ada perubahan pada `assets/integrity_policy.json`, inventaris 165 aset, tiga aset terlindungi (`ecm_database.json`, `revcore_profiles.json`, `revcore_maps/manifest.tsv`), pin sertifikat, atau logika Gate 10.
- Perubahan hanya pada source Kotlin; UI/DEX manifest dibangun ulang dan ditandatangani oleh prosedur Release resmi.
- Unit test integritas lulus (`IntegrityGateTest` 31, `GateStagingTest` 13).
- Verifikasi Gate 10 pada APK hasil build sudah dicatat di bagian 8; policy dan inventaris tidak diubah pada order ini.

---

## 8. APK Release resmi

Build dilakukan melalui prosedur resmi `tools/revcore-official-release-gui.ps1`. Password dimasukkan sendiri oleh Owner melalui dialog WinForms; executor tidak meminta, membaca, menyimpan, menampilkan, atau mencatat password. Status resmi: `SUCCESS 2026-11-22T22:09:32.5750822+07:00`.

| Item | Hasil terverifikasi |
|---|---|
| Lokasi APK | `dist/RevCore build 221126.apk` |
| Ukuran | 39.841.086 byte |
| SHA-256 APK | `cb5892a3b956c566e2f9cb3f0ae77f84488f1cd4902803b0ca756298720ba556` |
| Checksum file | `dist/RevCore build 221126.sha256.txt` — cocok dengan hasil `sha256sum` |
| Package / versi | `com.revcore.android` versionCode 1, versionName 1.0 |
| SDK | minSdk 26, targetSdk 36 |
| Integritas ZIP | `unzip -t` — No errors detected |
| Tanda tangan | `apksigner verify` — Verifies; v1 false, v2 true, v3 true |
| Sertifikat | SHA-256 `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| Subject sertifikat | `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta, ST=DKI Jakarta, C=ID` |
| Debuggable | tidak ada atribut `debuggable` |
| Material kunci | tidak ada `.p12/.jks/.keystore` atau direktori kunci di dalam APK |
| Build sebelumnya | diarsipkan ke `dist/archive/RevCore build 221126_220931.apk` |

**Verifikasi Gate 10 independen pada APK hasil build** (skrip read-only `delivery/rebuild_161126_20261122/verify_gate10.ps1`, dijalankan setelah build):

| Pemeriksaan | Hasil |
|---|---|
| Envelope policy | `format/payload/signature/algorithm/signerCertSha256`; `SHA256withRSA`, signature 344 byte |
| Payload policy | `debug=false`, package `com.revcore.android`, versionCode 1, versionName 1.0, schema 1, baud 10400 |
| Pin penanda tangan | `expectedSignerSha256` = `c649cb4f…18d098` — cocok dengan sertifikat APK |
| SHA-256 mentah `integrity_policy.json` | `2031223593248b42fcf1cb8c5935b025cdcb8c75849d814366e970d69c06cbfc` |
| Inventaris aset | 165 entri di policy = 165 file `assets/` di APK (di luar policy, UI manifest, dan `dexopt/`) |
| Kecocokan aset | 0 hilang, 0 mismatch ukuran/SHA-256 |
| Aset terlindungi (3) | `ecm_database.json` `7c7a2dd6…0b652a`; `revcore_maps/manifest.tsv` `189593bb…fdd41`; `revcore_profiles.json` `9c0be864…03da4` — semuanya cocok |
| UI/DEX manifest | `SHA256withRSA`, signature 344 byte, signer `c649cb4f…18d098` |
| `policyDigest` UI manifest | `2031223593248b42fcf1cb8c5935b025cdcb8c75849d814366e970d69c06cbfc` = SHA-256 mentah policy — cocok |
| Inventaris UI/DEX | 20 resource UI + 1 `classes.dex` + 1 struktur; 0 mismatch ukuran/SHA-256 terhadap APK |

**Catatan:** tidak ada commit, push, perubahan branch, ZIP distribusi, atau publikasi yang dilakukan. APK ini disiapkan untuk pengujian Owner; status penerimaan tetap milik Owner Otoy.

---

## 9. Bagian yang belum terbukti / batas status

- **Software tested:** kompilasi dan 457 unit test JVM lulus.
- **Integration tested:** tidak; tidak ada perangkat pada sesi ini.
- **Hardware not tested:** Live Data pada ECU nyata, perubahan O2 nyata, popup/reset/renew Keyless pada kendaraan, dan Gate 10 runtime di perangkat.
- **Reconstructed/analyzed:** prosedur Keyless tetap berbasis firmware/rekonstruksi (`RECONSTRUCTED_FROM_BINARY`); status model tetap `PROTOCOL_ANALYZED` / `FIELD_TEST_PENDING`. Tidak ada `FIELD_VERIFIED` baru.
- **Owner validation pending:** pengujian perangkat dan keputusan penerimaan akhir tetap milik Owner Otoy.

---

## 10. Batas kewenangan yang dipatuhi

Tidak dilakukan: Git commit, Git push, perubahan branch, publikasi GitHub, pembuatan ZIP distribusi, unggahan ke pihak lain, perubahan versi/identitas aplikasi, perubahan signing key, penggunaan debug key, pelemahan/penonaktifan Gate 10, pembacaan/penyimpanan/pencentakan password signing, atau penyajian APK Debug sebagai pengganti Release.

Password signing hanya dimasukkan Owner melalui dialog resmi; executor tidak membaca, mencatat, menyimpan, atau menampilkannya.
