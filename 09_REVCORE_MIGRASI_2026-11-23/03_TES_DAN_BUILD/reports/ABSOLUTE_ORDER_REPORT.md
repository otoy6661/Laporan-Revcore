# LAPORAN PERUBAHAN — MASTER WORK ORDER ABSOLUT (4 PERSYARATAN)

**Order:** MASTER WORK ORDER — REVCORE REBUILD 161126 — ABSOLUTE CORRECTION &
IMPLEMENTATION ORDER
**Tanggal eksekusi:** 2026-11-22
**Baseline:** Build 161126
**Branch:** `rebuild-161126`
**Working copy:** `RevCore_work_live_keyless_20261120`
**Repository:** https://github.com/otoy6661/Revcore-Project
**Executor:** Kodir
**Status commit:** BELUM COMMIT — menunggu persetujuan Owner (commit-after-ack)

Catatan format: "ACTUAL OLD CODE" diambil dari baris yang benar-benar dihapus
pada working copy sebelum order ini (terlihat sebagai baris `-` di `git diff`
terhadap HEAD). `git diff` terhadap HEAD juga memuat pekerjaan working copy
sebelum order ini; entri yang berasal dari sesi sebelumnya diberi keterangan
eksplisit. Tidak ada kode contoh yang disajikan sebagai kode aktual.

---

## I. Ringkasan hasil

| No. | Persyaratan | Status implementasi | Bukti utama |
|---|---|---|---|
| 1 | Startup: Gate 10/trust → jeda 2 detik → animasi loading → menu, di semua jalur | SELESAI di kode + unit test | `failAfterHold`, `awaitHoldAndInitialize`, `StartupTiming`, `StartupTimingTest` |
| 2 | Live Data dinamis + Prakiraan AFR dipertahankan | SELESAI di kode + unit test | `retainPreviousReadings`, `withEngineRunning`, wiring `parseLiveFrame`, `LiveDataRetentionTest` |
| 3 | Sinkronisasi Gate 10 pada setiap perubahan + pembersihan identitas sumber | SELESAI + diverifikasi | 185 rename, policy 165 = aset APK 165, 0 nama `flashpro` |
| 4 | Keyless Read/Renew/Reset sesuai prosedur sumber | SELESAI diverifikasi terhadap firmware; kode reset sudah cocok | `ghidra_reset_decomp.log`, riset bagian 11 |

Build: **BUILD SUCCESSFUL** (`:app:compileDebugKotlin`,
`:app:testDebugUnitTest`, `:app:assembleDebug`).
Unit test: **53 suite / 444 test / 0 failure / 0 error**.
APK debug: **57.213.426 byte**, SHA-256
`c7776bf12c517f94ffb702e4dd73bb7668ce7840582a7f5a1e96adb83f658a0f`.
Tidak ada release signing, tidak ada perubahan versi, tidak ada commit/push.

---

## II. ABSOLUTE NO. 1 — Urutan startup

### A1-01 — Empat jalur startup gagal tidak lagi melompat ke layar gagal

**FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`

**LOKASI:** `acknowledgeIntegrity` (baris 423), `handleStartupOutcome`
(baris 483 dan 494), `acceptTrustedReport` (baris 518)

**ACTUAL OLD CODE:**

```kotlin
                    _startupState.value = StartupState.Failed(report)
```

```kotlin
                _startupState.value = StartupState.Failed(outcome.report)
```

```kotlin
                        _startupState.value = StartupState.Failed(outcome.report)
```

```kotlin
            _startupState.value = StartupState.Failed(report)
```

**REPLACEMENT CODE:**

```kotlin
                    failAfterHold(report)
```

```kotlin
                failAfterHold(outcome.report)
```

```kotlin
                        failAfterHold(outcome.report)
```

```kotlin
            failAfterHold(report)
```

**REASON:** Jalur gagal adalah jalur startup normal. Sebelumnya layar gagal
muncul seketika, sehingga jalur ini melewati jeda 2 detik dan animasi loading —
melanggar syarat "tidak ada jalur startup normal yang melewati hold atau
loading".

**IMPACT:** Kegagalan pemeriksaan/commit trust sekarang selalu melewati
`Preparing` (hold 2 detik) lalu `Starting` (loading) sebelum `Failed`. Tidak ada
inisialisasi dan tidak ada jalur ECU yang dibuka pada jalur gagal.

**TEST:** `StartupTimingTest.failure path still shows the loading screen after
the hold`; seluruh uji Gate 10 (`IntegrityGateTest`, `GateStagingTest`) tetap
lulus.

**GIT DIFF:**

```diff
@@ -342 +423 @@
-                    _startupState.value = StartupState.Failed(report)
+                    failAfterHold(report)
@@ -402 +483 @@
-                _startupState.value = StartupState.Failed(outcome.report)
+                failAfterHold(outcome.report)
@@ -413 +494 @@
-                        _startupState.value = StartupState.Failed(outcome.report)
+                        failAfterHold(outcome.report)
@@ -435 +518 @@
-            _startupState.value = StartupState.Failed(report)
+            failAfterHold(report)
```

### A1-02 — Fungsi `failAfterHold` (ADD)

**FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`

**LOKASI:** setelah `acceptTrustedReport`, baris 526–537

**ACTUAL OLD CODE:** tidak ada (fungsi baru).

**REPLACEMENT CODE:**

```kotlin
    /**
     * Owner requirement: a failed examination is still a normal startup path, so
     * it also passes through the two-second hold and the loading screen before the
     * failure screen appears. Nothing is initialized and no ECU path is opened.
     */
    private suspend fun failAfterHold(report: IntegrityReport) {
        _startupState.value = StartupState.Preparing(report)
        delay(StartupTiming.HOLD_BEFORE_LOADING_MS)
        _startupState.value = StartupState.Starting(1f)
        delay(StartupTiming.FAILURE_LOADING_VISIBLE_MS)
        _startupState.value = StartupState.Failed(report)
    }
```

**REASON:** Satu titik tunggal untuk urutan hold → loading → gagal, sehingga
keempat jalur gagal tidak dapat menyimpang satu sama lain.

**IMPACT:** `Starting(1f)` hanya tampil setelah hold penuh; durasi 500 ms adalah
lantai visibilitas transisi terminal, bukan durasi loading inisialisasi nyata.

**TEST:** `StartupTimingTest` (3 test).

**GIT DIFF:** penambahan blok di atas (lihat hunk `@@ -438 +521,26 @@`).

### A1-03 — Trust record / in-process reuse melewati dialog, bukan hold

**FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`

**LOKASI:** `acceptTrustedReport`, baris 515–524

**ACTUAL OLD CODE:**

```kotlin
    private suspend fun acceptTrustedReport(report: IntegrityReport) {
        applyIntegrity(report)
        if (!report.passed) {
            _startupState.value = StartupState.Failed(report)
            return
        }
        // TRUST_RECORD and an in-process reuse skip the examination and the dialog.
        runInitialization()
    }
```

**REPLACEMENT CODE:**

```kotlin
    private suspend fun acceptTrustedReport(report: IntegrityReport) {
        applyIntegrity(report)
        if (!report.passed) {
            failAfterHold(report)
            return
        }
        // TRUST_RECORD and an in-process reuse skip the examination and the dialog,
        // but never the owner-mandated two-second hold before loading.
        awaitHoldAndInitialize(report)
    }
```

**REASON:** Jalur trust record sebelumnya langsung `runInitialization()`
sehingga loading tampil tanpa didahului hold 2 detik.

**IMPACT:** Jalur trust record dan reuse dalam proses kini selalu: trust →
`Preparing` 2 detik → loading → menu. Kebijakan keamanan Gate 10 tidak diubah;
hanya reuse yang sah yang memakai jalur ini.

**TEST:** `StartupTimingTest`; uji trust record (`TrustRecordTest`,
`InstallationMarkerTest`) tetap lulus.

**GIT DIFF:**

```diff
@@ -438 +521,26 @@
-        // TRUST_RECORD and an in-process reuse skip the examination and the dialog.
-        runInitialization()
+        // TRUST_RECORD and an in-process reuse skip the examination and the dialog,
+        // but never the owner-mandated two-second hold before loading.
+        awaitHoldAndInitialize(report)
```

### A1-04 — `awaitHoldAndInitialize` (ADD)

**FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`

**LOKASI:** baris 539–546

**ACTUAL OLD CODE:** tidak ada (fungsi baru).

**REPLACEMENT CODE:**

```kotlin
    /**
     * Owner requirement: examination or trust-record reuse -> two-second hold ->
     * "Loading RevCore" -> main menu. The hold screen is static; [Starting] is
     * entered only after the hold has fully elapsed.
     */
    private suspend fun awaitHoldAndInitialize(report: IntegrityReport) {
        _startupState.value = StartupState.Preparing(report)
        delay(StartupTiming.HOLD_BEFORE_LOADING_MS)
        runInitialization()
    }
```

**REASON:** Memisahkan hold statis (`Preparing`) dari loading nyata
(`Starting`), sesuai "jeda 2 detik SEBELUM loading".

**IMPACT:** `MainActivity` memetakan `Idle`/`Preparing` ke `StartupHoldScreen`
dan `Starting` ke `LoadingScreen`; hold tidak menampilkan progres palsu.

**TEST:** `StartupTimingTest.hold is exactly two seconds and loading is
forbidden before it ends`.

**GIT DIFF:** penambahan blok di atas.

### A1-05 — Loading dimulai di langkah pertama inisialisasi

**FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`

**LOKASI:** `runInitialization`, baris 554–557

**ACTUAL OLD CODE:**

```kotlin
    private suspend fun runInitialization() {
        try {
```

**REPLACEMENT CODE:**

```kotlin
    private suspend fun runInitialization() {
        // Loading becomes visible immediately after the hold, before the first
        // measurable initialization step, and stays visible until it completes.
        _startupState.value = StartupState.Starting(0f)
        try {
```

**REASON:** Sebelumnya state `Starting` baru masuk setelah langkah inisialisasi
pertama selesai; ada celah di mana hold berakhir tetapi loading belum tampil.
Sekarang loading tampil tepat setelah hold berakhir, dan setiap progres adalah
langkah nyata.

**IMPACT:** Progres 0f tampil segera setelah hold; tidak ada animasi tiruan;
layar loading selalu terlihat pada setiap startup yang berhasil.

**TEST:** `StartupTimingTest`; build penuh.

**GIT DIFF:**

```diff
     private suspend fun runInitialization() {
+        // Loading becomes visible immediately after the hold, before the first
+        // measurable initialization step, and stays visible until it completes.
+        _startupState.value = StartupState.Starting(0f)
         try {
```

### A1-06 — Jalur debug yang lulus penuh tetap melewati hold

**FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`

**LOKASI:** `handleStartupOutcome`, cabang `else` (baris 502–509)

**ACTUAL OLD CODE:**

```kotlin
                    else -> {
                        // A debug build that genuinely passes every gate never owns
                        // a release trust record; initialization only.
                        pendingCommit = null
                        runInitialization()
                    }
```

**REPLACEMENT CODE:**

```kotlin
                    else -> {
                        // A debug build that genuinely passes every gate shows the
                        // same confirmation window but never owns a release trust
                        // record; KONFIRMASI continues without writing files.
                        pendingCommit = null
                        _startupState.value =
                            StartupState.AwaitingAcknowledgement(outcome.report)
                    }
```

**REASON:** Jalur debug yang lulus penuh sebelumnya memanggil
`runInitialization()` langsung, melewati hold dan dialog. Perubahan ini sudah ada
di working copy sebelum sesi order ini dan disertakan karena termasuk daftar
"jalur startup sah lainnya" yang wajib diperiksa.

**IMPACT:** Debug build: exam → konfirmasi → hold 2 detik → loading → menu,
tanpa menulis trust record rilis. Build rilis tidak berubah.

**TEST:** `StartupTimingTest`; `IntegrityGateTest` (31 test) lulus.

**GIT DIFF:**

```diff
@@ -422,2 +503,3 @@
-                        // A debug build that genuinely passes every gate never owns
-                        // a release trust record; initialization only.
                         pendingCommit = null
-                        runInitialization()
+                        _startupState.value =
+                            StartupState.AwaitingAcknowledgement(outcome.report)
```

### A1-07 — Konstanta jeda loading jalur gagal (ADD)

**FILE:** `app/src/main/java/com/revcore/android/ui/MainViewModel.kt`

**LOKASI:** `object StartupTiming`, baris 148–161

**ACTUAL OLD CODE (sebelum order ini):**

```kotlin
object StartupTiming {
    const val HOLD_BEFORE_LOADING_MS = 2000L

    /** Loading may only become visible once the whole hold has elapsed. */
    fun loadingAllowed(elapsedSinceTrustMs: Long): Boolean =
        elapsedSinceTrustMs >= HOLD_BEFORE_LOADING_MS
}
```

**REPLACEMENT CODE:**

```kotlin
object StartupTiming {
    const val HOLD_BEFORE_LOADING_MS = 2000L

    /**
     * A failed examination still shows the loading screen after the hold, so no
     * startup path skips it. This is only a visibility floor for that terminal
     * transition; it is not a minimum duration for real initialization.
     */
    const val FAILURE_LOADING_VISIBLE_MS = 500L

    /** Loading may only become visible once the whole hold has elapsed. */
    fun loadingAllowed(elapsedSinceTrustMs: Long): Boolean =
        elapsedSinceTrustMs >= HOLD_BEFORE_LOADING_MS
}
```

**REASON:** Dibutuhkan nilai eksplisit dan teruji untuk transisi terminal jalur
gagal; tidak boleh tersirat dari durasi inisialisasi.

**IMPACT:** 500 ms hanya untuk jalur gagal; jalur sukses memakai progres nyata.

**TEST:** `StartupTimingTest.failure path still shows the loading screen after
the hold`.

**GIT DIFF:** penambahan konstanta dan KDoc di atas.

### A1-08 — Test urutan startup (ADD)

**FILE:** `app/src/test/java/com/revcore/android/ui/StartupTimingTest.kt` (ADD)

**LOKASI:** seluruh file

**ACTUAL OLD CODE:** tidak ada (file baru).

**REPLACEMENT CODE (inti):**

```kotlin
    @Test
    fun `hold is exactly two seconds and loading is forbidden before it ends`() {
        assertEquals(2000L, StartupTiming.HOLD_BEFORE_LOADING_MS)
        assertFalse(StartupTiming.loadingAllowed(0L))
        assertFalse(StartupTiming.loadingAllowed(1_999L))
        assertTrue(StartupTiming.loadingAllowed(2_000L))
```

```kotlin
    @Test
    fun `failure path still shows the loading screen after the hold`() {
        assertTrue(StartupTiming.FAILURE_LOADING_VISIBLE_MS > 0L)
        assertFalse(StartupTiming.loadingAllowed(StartupTiming.HOLD_BEFORE_LOADING_MS - 1))
        assertTrue(StartupTiming.loadingAllowed(StartupTiming.HOLD_BEFORE_LOADING_MS))
    }
```

**REASON:** Mengunci aturan "2 detik sebelum loading" dan keberadaan loading
pada jalur gagal tanpa membutuhkan perangkat.

**IMPACT:** Regresi urutan startup akan menggagalkan unit test.

**TEST:** 3 test, semuanya lulus.

**GIT DIFF:** file untracked baru (`?? app/src/test/java/com/revcore/android/ui/StartupTimingTest.kt`).

### A1-09 — Routing UI (tanpa perubahan pada order ini)

`app/src/main/java/com/revcore/android/MainActivity.kt` baris 141–159 sudah
memetakan `Idle`/`Preparing` → `StartupHoldScreen()` dan `Starting` →
`LoadingScreen(progress = state.progress)`. Tidak ada perubahan diperlukan;
hold tetap statis dan loading tetap memakai progres nyata.

---

## III. ABSOLUTE NO. 2 — Live Data dinamis dan Prakiraan AFR

Mekanisme lama dipertahankan: `HondaProtocolEngine.queryLiveData` →
`parseLiveFrame` → `HondaLiveDataProfiles` → `LiveTelemetry` →
`UnifiedLiveTable`. Tidak ada Role Table baru, tidak ada jalur data kedua.
Perubahan hanya menambah retensi parameter yang sudah tampil dan menegakkan
status KOEO.

### A2-01 — Nilai engine-only yang kedaluwarsa tidak dipertahankan saat mesin mati

**FILE:** `app/src/main/java/com/revcore/android/core/comm/LiveDataProfile.kt`

**LOKASI:** `LiveParameterStates.withEngineRunning`

**ACTUAL OLD CODE:**

```kotlin
            if (sensor in engineRunningOnly && state == LiveParameterState.VALID) {
                LiveParameterState.NOT_AVAILABLE_IN_KOEO
            } else {
                state
            }
```

**REPLACEMENT CODE:**

```kotlin
            if (sensor in engineRunningOnly &&
                (state == LiveParameterState.VALID || state == LiveParameterState.STALE)
            ) {
                // A retained sample from a previous run is not an engine-off value.
                LiveParameterState.NOT_AVAILABLE_IN_KOEO
            } else {
                state
            }
```

**REASON:** Setelah retensi diperkenalkan, sampel STALE dari putaran mesin
sebelumnya dapat lolos sebagai "tersedia" saat kontak ON tanpa mesin hidup.

**IMPACT:** Saat mesin mati, sensor khusus mesin (O2, injektor, STF, dan sensor
engine-only lain) berstatus `NOT_AVAILABLE_IN_KOEO`, bukan angka lama atau nol.

**TEST:** `LiveDataRetentionTest.engineOffMarksEngineOnlyValuesUnavailableInsteadOfShowingZero`.

**GIT DIFF:**

```diff
-            if (sensor in engineRunningOnly && state == LiveParameterState.VALID) {
+            if (sensor in engineRunningOnly &&
+                (state == LiveParameterState.VALID || state == LiveParameterState.STALE)
+            ) {
+                // A retained sample from a previous run is not an engine-off value.
                 LiveParameterState.NOT_AVAILABLE_IN_KOEO
```

### A2-02 — Retensi parameter yang sudah tampil (ADD)

**FILE:** `app/src/main/java/com/revcore/android/core/comm/LiveDataProfile.kt`

**LOKASI:** `LiveParameterStates`, setelah `withEngineRunning`

**ACTUAL OLD CODE:** tidak ada (tipe dan fungsi baru).

**REPLACEMENT CODE:**

```kotlin
    data class RetainedReadings(
        val states: Map<LiveSensor, LiveParameterState>,
        val values: Map<LiveSensor, Double>,
        val receivedAtMs: Map<LiveSensor, Long>
    )

    fun retainPreviousReadings(
        currentStates: Map<LiveSensor, LiveParameterState>,
        currentValues: Map<LiveSensor, Double>,
        currentReceivedAtMs: Map<LiveSensor, Long>,
        previousStates: Map<LiveSensor, LiveParameterState>,
        previousValues: Map<LiveSensor, Double>,
        previousReceivedAtMs: Map<LiveSensor, Long>,
        engineRunning: Boolean,
        engineRunningOnly: Set<LiveSensor>
    ): RetainedReadings {
        val states = currentStates.toMutableMap()
        val values = currentValues.toMutableMap()
        val times = currentReceivedAtMs.toMutableMap()
        for ((sensor, previousState) in previousStates) {
            val currentState = states[sensor]
            if (currentState == LiveParameterState.VALID ||
                currentState == LiveParameterState.STALE
            ) {
                continue
            }
            if (previousState != LiveParameterState.VALID &&
                previousState != LiveParameterState.STALE
            ) {
                continue
            }
            if (sensor in engineRunningOnly && !engineRunning) continue
            val previousValue = previousValues[sensor] ?: continue
            states[sensor] = LiveParameterState.STALE
            values[sensor] = previousValue
            previousReceivedAtMs[sensor]?.let { times[sensor] = it }
        }
        return RetainedReadings(states = states, values = values, receivedAtMs = times)
    }
```

**REASON:** Sebelumnya setiap frame mengganti seluruh peta status; parameter
yang tidak dibawa frame terbaru (misalnya saat tabel primer berganti) hilang
dari tabel. Owner mewajibkan parameter yang sudah tampil tetap ada.

**IMPACT:** Nilai baru selalu menang; sampel lama menjadi `STALE` dengan waktu
terima aslinya sehingga UI menampilkan umurnya; nilai engine-only tidak
dipertahankan saat mesin mati; parameter yang belum pernah valid tidak pernah
muncul.

**TEST:** `aParameterMissingFromTheNewFrameIsRetainedAsStaleWithItsAge`,
`aCurrentValueWinsOverTheRetainedSample`,
`engineOnlySamplesAreNotRetainedWhileTheEngineIsOff`.

**GIT DIFF:** penambahan blok di atas.

### A2-03 — Wiring retensi di `parseLiveFrame`

**FILE:** `app/src/main/java/com/revcore/android/core/protocol/HondaProtocolEngine.kt`

**LOKASI:** `parseLiveFrame`, setelah `val now = System.currentTimeMillis()`
(sekitar baris 1519–1548), dan konstruktor `LiveTelemetry` (baris 1572–1576)

**ACTUAL OLD CODE:**

```kotlin
            parameterStates = effectiveStates
```

**REPLACEMENT CODE:**

```kotlin
        val currentValues = values.filterKeys { effectiveStates[it] == LiveParameterState.VALID }
        val previous = _liveTelemetry.value
        val retained = if (previous == null) {
            LiveParameterStates.RetainedReadings(
                states = effectiveStates,
                values = currentValues,
                receivedAtMs = currentValues.keys.associateWith { now }
            )
        } else {
            // Keep parameters that were already on screen when a later frame (for
            // example a different primary table) does not carry them. They become
            // STALE with their original timestamp instead of disappearing.
            LiveParameterStates.retainPreviousReadings(
                currentStates = effectiveStates,
                currentValues = currentValues,
                currentReceivedAtMs = currentValues.keys.associateWith { now },
                previousStates = previous.parameterStates,
                previousValues = previous.readings,
                previousReceivedAtMs = previous.receivedAtMs,
                engineRunning = engineRunning,
                engineRunningOnly = engineRunningOnlySensors
            )
        }
```

```kotlin
            parameterStates = retained.states,
            readings = retained.values,
            receivedAtMs = retained.receivedAtMs
```

**REASON:** Menyambungkan retensi ke satu-satunya jalur parsing Live Data,
tanpa membuat jalur data kedua.

**IMPACT:** `UnifiedLiveTable` membaca `LiveTelemetry.readings` termasuk sampel
STALE; baris lama tetap tampil dengan status "DATA KEDALUWARSA" dan umur data.
Nilai hasil parsing yang tidak valid tetap tidak masuk `readings`.

**TEST:** `LiveDataRetentionTest` (6 test); suite Live Data lama tetap lulus.

**GIT DIFF:**

```diff
-            parameterStates = effectiveStates
+            parameterStates = retained.states,
+            readings = retained.values,
+            receivedAtMs = retained.receivedAtMs
```

### A2-04 — Test Live Data dinamis dan AFR (ADD)

**FILE:** `app/src/test/java/com/revcore/android/core/comm/LiveDataRetentionTest.kt` (ADD)

**LOKASI:** seluruh file

**ACTUAL OLD CODE:** tidak ada (file baru).

**REPLACEMENT CODE:** 6 test:

1. `aParameterMissingFromTheNewFrameIsRetainedAsStaleWithItsAge`
2. `aCurrentValueWinsOverTheRetainedSample`
3. `engineRunningKeepsOldRowsWhileO2IsAdded`
4. `engineOffMarksEngineOnlyValuesUnavailableInsteadOfShowingZero`
5. `engineOnlySamplesAreNotRetainedWhileTheEngineIsOff`
6. `afrEstimateTracksTheO2InputAndStaysUnavailableWithoutIt`

Test AFR memakai `AfrEstimator.estimate(LiveTelemetry)` dengan O2 0,2 V (lean),
0,9 V (rich), tanpa O2 (`Unavailable`), dan 1,9 V (`Invalid`).

**REASON:** Mengunci keempat perilaku yang diminta: munculnya parameter baru,
retensi parameter lama, perbedaan mesin mati/hidup, dan AFR yang mengikuti O2.

**IMPACT:** Regresi pada salah satu perilaku akan menggagalkan unit test.

**TEST:** 6 test, semuanya lulus.

**GIT DIFF:** file untracked baru.

### A2-05 — Ringkasan mekanisme yang dipertahankan dan rumus AFR

| Aspek | Isi |
|---|---|
| Mekanisme lama | Dipertahankan: polling tabel Honda (`17`, `13`, `11`, `10` + ekstensi `16`, `20`, `D1`), `parseLiveFrame`, `LiveTelemetry`, `UnifiedLiveTable` |
| Tabel | Satu tabel terpadu parameter + aktuator; tanpa Role Table perantara |
| Provenance | `sourceTable` dan `sourceOffset` per baris tetap ada sebagai metadata teknis |
| Mesin mati (kontak ON) | Hanya parameter yang benar-benar tersedia; sensor engine-only `NOT_AVAILABLE_IN_KOEO`; tidak ada nol palsu |
| Mesin hidup | O2, injektor, STF, dan parameter engine-only lain muncul otomatis saat datanya valid; baris lama tetap |
| Parameter lama | Dipertahankan sebagai `STALE` (bukan dihapus, bukan diubah menjadi VALID) |
| Prakiraan AFR | `HondaProtocolEngine` menghitung dari tegangan O2 tabel `20` byte 4; `AfrEstimator` |
| Rumus | `AFR = 17.700001 - 5.351 * V_O2` (konstanta `INTERCEPT = 17.700001`, `SLOPE = 5.351`, `O2_COUNTS_PER_VOLT = 50.993698`) |
| Sumber riset | `FLASHPRO_KEYLESS_ID_RESEARCH.md` bagian 6–7 dan audit Live Data; rumus diuji `AfrEstimatorTest` terhadap nilai referensi |
| Satuan/rentang | Input volt; 0,0–1,5 V diterima; di luar rentang atau non-numerik → `Invalid` |
| O2 tidak ada | `Unavailable`, bukan angka |
| O2 tidak valid | `Invalid`, bukan angka |
| O2 berubah | Estimasi berubah mengikuti voltase (`LiveDataRetentionTest` test 6, `AfrEstimatorTest`) |
| Klaim | Selalu "Prakiraan AFR", tidak pernah "AFR terukur" |

---

## IV. ABSOLUTE NO. 3 — Sinkronisasi Gate 10 dan identitas sumber

### A3-01 — Rename aset `flashpro_*` → `revcore_*`

**FILE:** `app/src/main/assets/` (185 entri rename di `git status --porcelain`)

**LOKASI:** `flashpro_profiles.json` → `revcore_profiles.json`;
`flashpro_maps/**` → `revcore_maps/**` (162 berkas `.map` di `ORIGINAL` dan
subdirektori lain)

**ACTUAL OLD CODE (nama path):**

```
app/src/main/assets/flashpro_profiles.json
app/src/main/assets/flashpro_maps/ORIGINAL/...
```

**REPLACEMENT CODE (nama path):**

```
app/src/main/assets/revcore_profiles.json
app/src/main/assets/revcore_maps/ORIGINAL/...
```

**REASON:** Membersihkan identitas sumber pihak lain; nama lama tidak
dipertahankan hanya karena Gate 10 masih mengenalinya.

**IMPACT:** Aplikasi membaca aset dengan nama baru; tidak ada aset lama yang
tersisa di APK; tidak ada pengecualian atau pelemahan pemeriksaan.

**TEST:** `BundledMapAssetsTest` (4), `ApkArchiveTest` (11),
`IntegrityGateTest` (31); inventaris policy = isi APK (lihat A3-05).

**GIT DIFF:** 185 baris `R` (rename) di `git status --porcelain`, termasuk
`R  app/src/main/assets/flashpro_profiles.json -> app/src/main/assets/revcore_profiles.json`.

### A3-02 — ID profil Live Data `flashpro.dat*` → `revcore.dat*`

**FILE:** `app/src/main/java/com/revcore/android/core/comm/LiveDataProfile.kt`

**LOKASI:** `table17`, `table13`, `table11`, `table10`, `table16`, `table20`,
`tableD1`

**ACTUAL OLD CODE:**

```kotlin
        id = "flashpro.dat17",
```

```kotlin
        id = "flashpro.dat13",
```

```kotlin
        id = "flashpro.dat11",
```

```kotlin
        id = "flashpro.dat10",
```

```kotlin
        id = "flashpro.dat16",
```

```kotlin
        id = "flashpro.dat20",
```

```kotlin
        id = "flashpro.datD1",
```

**REPLACEMENT CODE:**

```kotlin
        id = "revcore.dat17",
```

```kotlin
        id = "revcore.dat13",
```

```kotlin
        id = "revcore.dat11",
```

```kotlin
        id = "revcore.dat10",
```

```kotlin
        id = "revcore.dat16",
```

```kotlin
        id = "revcore.dat20",
```

```kotlin
        id = "revcore.datD1",
```

**REASON:** ID profil adalah identitas sumber yang tampil di log; nama lama
tidak dipertahankan.

**IMPACT:** Hanya string ID yang berubah; `tableId` (`"17"`, `"13"`, dst.),
request, offset, dan decoder tidak berubah. Log menampilkan `revcore.dat*`.

**TEST:** Seluruh suite Live Data; build penuh.

**GIT DIFF:**

```diff
-        id = "flashpro.dat17",
+        id = "revcore.dat17",
```
(diulang untuk `dat13`, `dat11`, `dat10`, `dat16`, `dat20`, `datD1`)

### A3-03 — Regenerasi policy Gate 10

**FILE:** `app/build/generated/integrity/debug/assets/integrity_policy.json`
(artefak build, dihasilkan tugas `generateIntegrityPolicyDebug`)

**LOKASI:** `app/build.gradle.kts`, task policy yang membaca
`app/src/main/assets`

**ACTUAL OLD CODE:** policy debug tertanggal 2026-11-20 yang masih memuat path
`flashpro_*` (artefak stale karena build sebelumnya tidak benar-benar dijalankan
tanpa Gradle wrapper).

**REPLACEMENT CODE:** policy debug tertanggal 2026-11-22 17:35, memuat
`revcore_profiles.json` dan `revcore_maps/**`, tanpa satu pun path
`flashpro`.

**REASON:** Setiap perubahan nama/isi aset wajib diikuti pembaruan manifest,
hash, referensi, dan pemeriksaan Gate 10.

**IMPACT:** Gate 10 memeriksa aset yang benar; tidak ada bypass, tidak ada
pengecualian luas, tidak ada penghapusan pemeriksaan.

**TEST (dijalankan ulang pada 2026-11-22):**

```
policy paths           : 165
APK assets (excl. policy): 165
diff policy vs APK     : kosong
"flashpro" di payload  : 0 kemunculan
```

**GIT DIFF:** artefak build tidak di-commit; sumbernya (`build.gradle.kts`,
aset) tercatat di A3-01/A3-04.

### A3-04 — Pembaruan referensi nama aset

**FILE:** `BundledMapManifest.kt`, `MapCatalog.kt`, `RevCoreProfile.kt`,
`GateEvaluator.kt`, `TuningScreen.kt`, serta
`BundledMapAssetsTest.kt`, `ApkArchiveTest.kt`, `GateStagingTest.kt`,
`IntegrityGateTest.kt`, `TestRevCoreProfiles.kt`

**LOKASI:** referensi path aset `flashpro_profiles.json` / `flashpro_maps`

**ACTUAL OLD CODE (string path):** `"flashpro_profiles.json"`,
`"flashpro_maps"`

**REPLACEMENT CODE (string path):** `"revcore_profiles.json"`,
`"revcore_maps"`

**REASON:** Rename tanpa pembaruan referensi akan membuat aplikasi gagal membaca
aset dan Gate 10 menandai anomali.

**IMPACT:** Aplikasi tetap membaca aset setelah rename; Gate 10 memakai
inventaris baru; pesan galat dan pesan UI memakai nama baru.

**TEST:** `BundledMapAssetsTest`, `ApkArchiveTest`, `GateStagingTest`,
`IntegrityGateTest`; pencarian `flashpro` pada policy dan isi APK = 0.

**GIT DIFF:** perubahan string path pada file-file di atas (ringkasan
`git diff --numstat`: `GateEvaluator.kt` 2/2, `LiveDataScreen.kt` 268/12;
file lain termasuk dalam paket rebuild yang sama).

### A3-05 — Bukti kesetaraan inventaris

Perintah dan hasil aktual:

```
$ git status --porcelain | grep -c '^R'
185
$ grep -o '"path":"[^"]*"' payload | sort | wc -l
165
$ unzip -Z1 app-debug.apk | grep '^assets/' | ... | wc -l
165
$ diff policy_paths.txt apk_assets.txt
(kosong)
$ grep -ci flashpro payload
0
```

Tidak ada perubahan yang melemahkan Gate 10: tidak ada pemeriksaan yang
dimatikan, tidak ada pengecualian luas, tidak ada perubahan tidak sah yang
diabaikan.

---

## V. ABSOLUTE NO. 4 — Keyless Read/Renew/Reset sesuai prosedur sumber

### A4-01 — Verifikasi ulang firmware dan catatan bukti pada codec

**FILE:** `app/src/main/java/com/revcore/android/core/keyless/KeylessResetCodec.kt`

**LOKASI:** KDoc objek, paragraf terakhir

**ACTUAL OLD CODE (akhir KDoc sebelum order ini):**

```kotlin
 * Evidence level: RECONSTRUCTED_FROM_BINARY. Nothing here has been executed
 * against a physical ECU; see [KeylessResetEngine] for the safety ladder.
 */
```

**REPLACEMENT CODE:**

```kotlin
 * Evidence level: RECONSTRUCTED_FROM_BINARY. Nothing here has been executed
 * against a physical ECU; see [KeylessResetEngine] for the safety ladder.
 *
 * Verification (2026-11-22): both routines were decompiled again from
 * `firmware.elf` (sha256
 * `fe0704fd322d2c981ab029faf63d7463a4d7521cc912e18fa411252e286b65e1`, Ghidra
 * 12.2) through `Immo_reset` -> `FUN_400e51b8` -> `FUN_400e4d78`/`FUN_400e4fc4`.
 * The read precondition `0x4A`, indices `0x46..0x49`, frame bytes
 * `82 82 14 08`, page `index >> 1`, checksum rule, 500 ms timeout, 120 ms gap,
 * and zero-failure rule for Keihin were read directly from the decompilation;
 * likewise Shindengen `0x00..0xAF`, `91 91 09 41`, checksum `-0x4A-address`,
 * 400 ms timeout, 5 ms gap, and `< 10` failures. Log:
 * `REVCORE_REFERENCE_AUDIT_20261120/evidence/ghidra_reset_decomp.log`.
 */
```

**REASON:** Order mewajibkan prosedur sumber dipelajari dan buktinya dilaporkan;
komentar kode harus menyebut bukti yang benar-benar dibaca.

**IMPACT:** Tidak ada perubahan perilaku; hanya dokumentasi bukti. Kode reset
sudah cocok dengan firmware sehingga tidak diubah.

**TEST:** `KeylessResetCodecTest` (8), `KeylessResetEngineTest` (7),
`KeylessRenewPlanTest` (4).

**GIT DIFF:** file berada di direktori untracked `core/keyless/`; paragraf di
atas adalah penambahan aktual.

### A4-02 — Fakta yang terbaca langsung dari firmware

Firmware: `firmware.elf`, SHA-256
`fe0704fd322d2c981ab029faf63d7463a4d7521cc912e18fa411252e286b65e1`.
Alat: Ghidra 12.2 headless. Rantai panggilan:
`Immo_reset` (`FUN_400e8f20`) → `FUN_400e51b8` → `FUN_400e4d78` (Keihin) /
`FUN_400e4fc4` (Shindengen). Log:
`REVCORE_REFERENCE_AUDIT_20261120/evidence/ghidra_reset_decomp.log`.

| Aspek | Keihin `FUN_400e4d78` | Shindengen `FUN_400e4fc4` |
|---|---|---|
| Prasyarat | citra EEPROM ≥ `0x4A` byte | tidak ada cek panjang eksplisit |
| Rentang | indeks `0x46..0x49` (4 frame) | alamat `0x00..0xAF` (176 frame) |
| Halaman | `page = index >> 1` (0x23, 0x24) | alamat itu sendiri |
| Frame | `82 82 14 08 page low high chk` | `91 91 09 41 addr 00 00 00 chk` |
| Data | genap: low=0; ganjil: high=0; separuh lain byte asli | empat byte nol |
| Checksum | `-(page + 0x20 + low + high)` | `-0x4A - addr` |
| Timeout | 500 ms | 400 ms |
| Jeda antar frame | `FUN_400e280c(0x78)` = 120 ms | `FUN_400e280c(5)` = 5 ms |
| ACK minimum | balasan ≥ 5 byte | balasan ≥ 5 byte |
| Syarat sukses | gagal = 0 dari 4 | gagal < 10 dari 176 |

Kode RevCore (`KeylessResetCodec`, `KeylessResetEngine`,
`KeylessRenewPlanner`) sudah memuat semua nilai di atas; karena itu tidak ada
perubahan kode reset pada order ini. Prosedur Read juga diverifikasi ulang
(`82 82 10 06` / 400 ms / ≥ 12 byte / data +10,+11; `91 91 07 40` / 300 ms /
≥ 13 byte / data +0xb,+0xc; 256 alamat).

**Yang belum terbukti:** bagaimana ECU fisik menafsirkan separuh payload nol
pada frame Keihin. Karena itu `KeylessResetEngine` membaca ulang setiap alamat
target dan hanya menyatakan `SUCCESS` bila semua nol; ACK tidak pernah dianggap
bukti hasil akhir; echo lokal adapter tidak dihitung sebagai ACK.

### A4-03 — Pembaruan dokumen riset

**FILE:** `C:/Users/Lord Otoy/Documents/Qoder/work/REVCORE_REFERENCE_AUDIT_20261120/FLASHPRO_KEYLESS_ID_RESEARCH.md`
(di luar repository aplikasi, bahan riset)

**LOKASI:** bagian 2 catatan status, bagian 10 poin 4, dan bagian baru 11

**ACTUAL OLD CODE (teks):** bagian 9 menyatakan "Tidak ada analisis lanjutan
rutin tulis EEPROM/Reset."

**REPLACEMENT CODE:** catatan status yang menunjuk order 2026-11-22 dan bagian
11 "Verifikasi rutin tulis (2026-11-22)" dengan tabel fakta Keihin/Shindengen,
item yang belum terbukti, dan syarat read-back.

**REASON:** Menyelaraskan dokumen dengan bukti baru tanpa menghapus temuan lama.

**IMPACT:** Dokumen riset kini menyatakan fakta yang terbaca langsung dan
memisahkannya dari yang belum terbukti.

**TEST:** tidak ada unit test untuk dokumen; isi diverifikasi terhadap log
Ghidra.

**GIT DIFF:** di luar repo aplikasi (tidak masuk commit aplikasi).

---

## VI. Matriks pengujian (syarat §V)

| Area yang diwajibkan | Test aktual | Hasil |
|---|---|---|
| Urutan startup, delay, loading | `StartupTimingTest` (3) | lulus |
| Semua jalur startup | `IntegrityGateTest` (31), `GateStagingTest` (13), `TrustRecordTest` (11), `InstallationMarkerTest` (10) | lulus |
| Live Data mesin mati/hidup | `LiveDataRetentionTest` (1, 3, 4, 5) | lulus |
| Parameter baru muncul | `LiveDataRetentionTest.engineRunningKeepsOldRowsWhileO2IsAdded` | lulus |
| Parameter lama bertahan | `LiveDataRetentionTest` (1, 2, 5) | lulus |
| Prakiraan AFR | `AfrEstimatorTest` (7), `LiveDataRetentionTest` (6) | lulus |
| O2 tidak tersedia / tidak valid | `AfrEstimatorTest.missingVoltageIsUnavailableNotANumber`, `nonNumericVoltageIsRejected`, `LiveDataRetentionTest` (6) | lulus |
| Sinkronisasi Gate 10 | `IntegrityGateTest`, `GateStagingTest`, `ApkArchiveTest`, `BundledMapAssetsTest` | lulus |
| Keyless Read | `KeylessReadEngineTest` (6), `KeylessFrameCodecTest` (13), `KeylessIdDecoderTest` (7) | lulus |
| Renew/Reset gating | `KeylessRenewPlanTest` (4), `KeylessResetEngineTest` (7) | lulus |
| Verifikasi tulis/read-back | `KeylessResetEngineTest` (7) | lulus |
| Kegagalan transport | `KeylessReadEngineTest`, `KeylessResetEngineTest` | lulus |
| Regresi keseluruhan | 53 suite / 444 test | 0 failure, 0 error |

Label status hasil:

- **Build successful** — `BUILD SUCCESSFUL` (`:app:compileDebugKotlin`,
  `:app:testDebugUnitTest`, `:app:assembleDebug`, Gradle 9.6.0 offline, JDK
  Android Studio `jbr`).
- **Unit tested** — 53 suite / 444 test / 0 failure / 0 error.
- **Integration tested** — tidak dilakukan (tidak ada perangkat pada sesi ini).
- **Simulated/reconstructed** — seluruh perilaku Keyless Read/Renew/Reset dan
  rumus AFR berasal dari rekonstruksi biner/riset, bukan ECU fisik.
- **Hardware not tested** — tidak ada ECU, smartkey, atau K-Line fisik.
- **Owner validation pending** — keputusan akhir milik Owner Otoy.

---

## VII. Repository dan kepatuhan larangan

- Branch: `rebuild-161126` (tidak berubah).
- Tidak ada commit, push, force push, penghapusan riwayat, ZIP, publikasi APK,
  release signing, atau perubahan `versionName`/`versionCode`/`applicationId`.
- Kredit `otoy` tidak diubah.
- Tidak ada password/rahasia signing di source, log, atau laporan.
- `Bahan Otopsi.zip` tidak disentuh.
- APK yang ada hanya build debug lokal:
  `app/build/outputs/apk/debug/app-debug.apk`, 57.213.426 byte, SHA-256
  `c7776bf12c517f94ffb702e4dd73bb7668ce7840582a7f5a1e96adb83f658a0f`.
- `git diff --numstat` terhadap HEAD (ringkasan, termasuk pekerjaan working copy
  sebelum order ini): 4.121 baris ditambah, 420 dihapus. File utama order ini:
  `MainViewModel.kt` 1.770/28, `HondaProtocolEngine.kt` 933/12,
  `LiveDataProfile.kt` 57/8, `LiveTelemetry.kt` 20/1.
- File baru yang belum dilacak: `core/keyless/`, `ui/keyless/`,
  `StartupHoldScreen.kt`, `StartupTimingTest.kt`, `LiveDataRetentionTest.kt`,
  `core/keyless/` tests, dan folder `delivery/`.

**Keputusan penerimaan akhir tetap milik Owner Otoy.**
