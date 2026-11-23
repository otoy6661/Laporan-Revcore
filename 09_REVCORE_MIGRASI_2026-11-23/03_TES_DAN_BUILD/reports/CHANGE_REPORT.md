# REVCORE REBUILD 161126 — CHANGE REPORT (§V FORMAT)

Order: `MASTER WORK ORDER — REVCORE REBUILD: ROLLBACK BUILD 161126 + IMPLEMENTASI 10 TUGAS TERPADU`
Owner: Otoy Sullivan (`otoy`) — Executor: Kodir
Date: 2026-11-22
Baseline: Build 161126 (commit `656bed2 rollback: restore app source to Build 161126`, documented in `1cc9120`)
Branch: `rebuild-161126`
Package: `com.revcore.android` (unchanged)

Every entry below follows: FILE → ACTUAL OLD CODE → REPLACEMENT CODE → REASON → IMPACT → TEST → GIT DIFF.
Entries are grouped by the ten ordered tasks. `git diff` counts are working-tree changes against the 161126 baseline (uncommitted).

---

## TASK 1 — Unified Live Data table

### 1.1 New unified table model
- **FILE**: `app/src/main/java/com/revcore/android/core/comm/UnifiedLiveTable.kt` (new)
- **ACTUAL OLD CODE**: none (no unified table existed; parameters and actuator rows were rendered by separate ad-hoc sections).
- **REPLACEMENT CODE**:
```kotlin
data class UnifiedLiveRow(
    val key: String,
    val labelId: String,
    val labelEn: String,
    val valueText: String,
    val unit: String,
    val state: LiveRowState,
    val tableId: Int?,
    val payloadIndex: Int?,
    val evidence: String
)

object UnifiedLiveTable {
    fun build(telemetry: LiveTelemetry, now: Long = System.currentTimeMillis()): List<UnifiedLiveRow> { ... }
}
```
- **REASON**: one table for parameter + actuator, no separate "Role Table" layer; ECU mapping is carried per row (`tableId`, `payloadIndex`) so every value is traceable.
- **IMPACT**: `LiveDataScreen` renders one `UnifiedLiveTableCard`; invalid/unsupported values render as status text, never as numbers.
- **TEST**: `UnifiedLiveTableTest` (labels, traceability, invalid rows never expose a value, actuator inclusion).
- **GIT DIFF**: new file; `LiveDataScreen.kt` +268/−12 (see `git diff --numstat`).

### 1.2 Single table in Live Data UI
- **FILE**: `app/src/main/java/com/revcore/android/ui/screens/LiveDataScreen.kt`
- **ACTUAL OLD CODE**: separate parameter block plus conditional AFR section; no unified traceable row list.
- **REPLACEMENT CODE**:
```kotlin
val unifiedRows = UnifiedLiveTable.build(telemetry)
...
// Single unified table: live parameters and actuators share one list.
// Rows come from the reconstructed table definitions, so every value is
// traceable to the table id and payload offset that produced it.
UnifiedLiveTableCard(rows = unifiedRows, isId = isId)
```
- **REASON**: task 1 explicitly forbids a new Role Table layer and requires one traceable table.
- **IMPACT**: one list; rows show status (`VALID`, `STALE`, `UNSUPPORTED`, `INVALID`, …) derived from `LiveTelemetry`.
- **TEST**: `UnifiedLiveTableTest`; compile of `LiveDataScreen`.
- **GIT DIFF**: `LiveDataScreen.kt | 268 ++-` (shared with 1.2 and task 2).

---

## TASK 2 — "Prakiraan AFR" from researched formula

- **FILE**: `app/src/main/java/com/revcore/android/core/comm/AfrEstimator.kt` (new); `LiveDataScreen.kt`
- **ACTUAL OLD CODE**:
```kotlin
// AFR is reported only if the ECU itself published it. The reconstructed
// Honda tables carry a narrowband O2 voltage, so AFR stays unsupported
// and says so instead of showing a host-side conversion.
```
- **REPLACEMENT CODE**:
```kotlin
object AfrEstimator {
    const val LABEL_INDONESIAN = "Prakiraan AFR"
    const val LABEL_ENGLISH = "Estimated AFR"
    fun estimate(telemetry: LiveTelemetry): Result { ... }   // Estimated | Unavailable | Invalid
}
...
// "Prakiraan AFR" is always present: the component stays visible even
// when the O2 input is missing or rejected, showing the reason instead
// of a fabricated number. The value is never called measured AFR.
AfrEstimateCard(estimate = afrEstimate, telemetry = telemetry, isId = isId)
```
- **REASON**: AFR component must always be present and must never be presented as measured AFR; no invented constants.
- **IMPACT**: label/caveat always visible; `Estimated` only when a valid narrowband O2 voltage exists; otherwise `Unavailable`/`Invalid` with reason; source line names the O2 table value used.
- **TEST**: `AfrEstimatorTest` (estimated vs unavailable vs invalid, caveat text present).
- **GIT DIFF**: new file `AfrEstimator.kt`; `LiveDataScreen.kt` additions shown above.

---

## TASK 3 — Log fixes (real `.txt` share, honest "Buka Lokasi", structured events)

### 3.1 Share a real saved `.txt`
- **FILE**: `app/src/main/java/com/revcore/android/ui/screens/LogScreen.kt`; `MainViewModel.createLogShareFile`
- **ACTUAL OLD CODE**:
```kotlin
val text = viewModel.exportLogAsText()
val logFileName = viewModel.generateLogFileName()
val sendIntent = Intent().apply {
    action = Intent.ACTION_SEND
    putExtra(Intent.EXTRA_TEXT, text)
    ...
}
context.startActivity(Intent.createChooser(sendIntent, logFileName))
```
- **REPLACEMENT CODE**:
```kotlin
shareLogFile(context, viewModel, isIndonesian)
...
fun createLogShareFile(context: Context, fileName: String = generateLogFileName()): Pair<Uri?, String> {
    val file = File(shareDir, fileName)
    file.writeText(exportLogAsText(), Charsets.UTF_8)
    if (!file.isFile || file.length() <= 0L) return Pair(null, "GAGAL: File log berbagi kosong atau tidak tersimpan.")
    val uri = FileProvider.getUriForFile(context, "${context.packageName}.fileprovider", file)
    ...
}
```
- **REASON**: sharing used to send only text; the task requires sharing the real saved `.txt` file and never claiming success when the write failed.
- **IMPACT**: shares a genuine file via `FileProvider` (`res/xml/file_paths.xml`, authority `${applicationId}.fileprovider`); empty/failed write returns an honest error and no chooser.
- **TEST**: compile + `:app:assembleDebug`; provider declared in manifest; no UI change beyond the share action.
- **GIT DIFF**: `LogScreen.kt | 224 ++-`; `MainActivity`/manifest additions.

### 3.2 "Buka Lokasi" never reports success on failure
- **FILE**: `LogScreen.kt`
- **ACTUAL OLD CODE**:
```kotlin
val intent = RevCoreStorage.openFolderIntent(RevCoreStorage.Folder.LOGS)
if (intent != null) { context.startActivity(intent) }
else { Toast.makeText(context, "Folder tidak dapat dibuka ...", ...).show() }
```
- **REPLACEMENT CODE**:
```kotlin
for (intent in RevCoreStorage.folderViewIntents(RevCoreStorage.Folder.LOGS)) {
    if (launchGuarded(context, intent)) return
}
if (launchGuarded(context, RevCoreStorage.folderPickerIntent(RevCoreStorage.Folder.LOGS))) return
Toast.makeText(context, "Tidak ada aplikasi yang dapat membuka folder ini. File log tetap tersimpan di Internal Storage/Revcore/Logs.", Toast.LENGTH_LONG).show()
```
- **REASON**: `startActivity` can throw or resolve to a handler that rejects the URI; the old code assumed success.
- **IMPACT**: each candidate intent is resolved and started inside try/catch; the final fallback is an explicit failure message, never a success claim.
- **TEST**: compile; manual path not hardware-dependent (owner validation pending on device file managers).

### 3.3 Structured event log per menu
- **FILES**: `MainViewModel.kt`
- **ACTUAL OLD CODE**: partial/inconsistent events; several operations ended without a start or completion record.
- **REPLACEMENT CODE** (statuses now emitted):
```text
Info Kendaraan : ECU_IDENTIFY_STARTED / ECU_IDENTIFIED / ECU_IDENTIFY_FAILED
Live Data      : LIVE_DATA_STARTED / LIVE_DATA_STALLED / LIVE_DATA_STOPPED (suspended flag in detail)
DTC            : DTC_SCAN_STARTED / DTC_CLEAR_STARTED / DTC_CLEAR_<STATUS> / DTC_SCAN_FAILED / DTC_CLEAR_FAILED
Keyless        : KEYLESS_ID_READ_STARTED / KEYLESS_ID_READ_COMPLETED / KEYLESS_ID_READ_FAILED
                 KEYLESS_RESET_CONFIRMED / KEYLESS_<ACTION>_<OUTCOME> / KEYLESS_RESET_FAILED / KEYLESS_RESET_INTERRUPTED / KEYLESS_RESET_REFUSED
Tuning/Remap   : TuningLogStore.begin/finish (connect, read start, write start, complete/fail)
                 REMAP_READ_<OUTCOME> / REMAP_READ_FAILED / REMAP_WRITE_<OUTCOME> / REMAP_WRITE_FAILED
                 REMAP_VERIFY_<OUTCOME> / REMAP_REFUSED_<REASON> / REMAP_WRITE_CANCELLED_RECOVERY_REQUIRED
USB/link       : USB_ATTACHED / USB_DETACHED / USB_PERMISSION_REQUEST / USB_PERMISSION_DENIED / USB_CONNECTED
                 MANUAL_DISCONNECT / ADAPTER_LOST (engine)
```
- **REASON**: task 3 requires structured per-menu events and forbids recording success before the operation actually succeeds.
- **IMPACT**: success statuses are written only after verified outcomes (`KeylessProcessOutcome.SUCCESS`, `ClearDtcStatus.SUCCESS_VERIFIED`, `RemapOutcome.SUCCESS`, `dbMatch && compatibilityVerified`); failures carry error details.
- **TEST**: existing `TuningLogStoreTest` + compile; event strings reviewed by grep.
- **GIT DIFF**: `MainViewModel.kt | 1717 ++-` (all tasks combined; see individual sections).

---

## TASK 4 — Full Keyless ID

### 4.1 Inputs, read, outcome and UI
- **FILES**: `core/keyless/*` (new: models, profile resolver, frame codec, ID decoder, read engine, renew plan, reset codec/engine), `ui/keyless/*` (new), `ui/screens/KeylessScreen.kt` (new), `HondaProtocolEngine.runKeylessRead/runKeylessReset`, `MainViewModel.startKeylessRead/startKeylessReset`.
- **ACTUAL OLD CODE**: no Keyless screen or engine in the 161126 baseline.
- **REPLACEMENT CODE** (core flow):
```kotlin
val report = protocolEngine.runKeylessRead(model) { onKeylessProgress(it) }
if (!isActive) return@launch
_keylessState.update { it.copy(report = report, ...) }
```
```kotlin
val report = protocolEngine.runKeylessReset(model = model, action = action, confirmed = true) { ... }
```
- **REASON**: read/status/Renew/Reset with failure handling, Protocol Manager lease, event log, USB preconditions.
- **IMPACT**: success requires decoded ID from read-back-verified frames, not merely a communication response.
- **TEST**: `KeylessFrameCodecTest`, `KeylessIdDecoderTest`, `KeylessProfileResolverTest`, `KeylessReadEngineTest`, `KeylessRenewPlanTest`, `KeylessResetCodecTest`, `KeylessResetEngineTest`.
- **GIT DIFF**: new packages; `MainViewModel.kt` keyless sections.

### 4.2 Renew/Reset only after a verified ID read
- **FILES**: `MainViewModel.teardownAdapter`, `KeylessScreen.kt`
- **ACTUAL OLD CODE**: reset gate existed only inside `startKeylessReset`; buttons were enabled on connection + model, and a report from a closed session could survive a disconnect.
- **REPLACEMENT CODE**:
```kotlin
// MainViewModel.teardownAdapter
// The verified ID read is evidence from the session that just ended.
// Renew/Reset requires a fresh read after the adapter is reopened.
_keylessState.update { it.copy(report = null) }
```
```kotlin
// KeylessScreen.kt
enabled = isConnected && !state.busy && selectedModel != null && state.report?.success == true
...
if (state.report?.success != true) {
    Text("Baca Keyless ID hingga berhasil sebelum RESET/RENEW diaktifkan.", ...,
        modifier = Modifier.testTag("keyless_reset_requires_read"))
}
```
- **REASON**: "Renew/Reset hanya setelah ID berhasil diverifikasi"; a communication response alone is not proof of a read.
- **IMPACT**: stale evidence is discarded on teardown; buttons stay disabled with an explicit hint until a successful read; the ViewModel refusal (`read_required`) remains as a second line of defence.
- **TEST**: compile + existing keyless tests; `startKeylessReset` guard already covered by engine tests; owner device validation pending.

---

## TASK 5 — Tuning/Remap overhaul

### 5.1 Engine + guard + UI (wired actions only)
- **FILES**: `core/remap/RemapEngine.kt`, `RemapCodec.kt`, `RemapChecksum.kt`, `RemapSeedKey.kt`, `RemapVendorStrategies.kt` (new); `data/maps/RemapExecutionGuard.kt` (new); `ui/RemapOperationState.kt` (new); `TuningScreen.kt` (`RemapExecutionCard`, `RemapWorkflowCard`).
- **ACTUAL OLD CODE**: baseline had no remap execution engine or guard.
- **REPLACEMENT CODE** (write path is not auto-verified by itself):
```kotlin
suspend fun writeImage(...) // every block ACKed; SUCCESS here means ACK, not memory proof
suspend fun verifyImage(...) // separate read-back verification pass
```
```kotlin
onRead = { viewModel.startRemapRead() },
onVerify = { viewModel.startRemapVerify() },
onWrite = { remapConfirmText = ""; showRemapDialog = true },
onQueryFlashCount = { viewModel.queryRemapFlashCount() },
onResetEcm = { remapConfirmText = ""; showRemapResetConfirm = true },
onCancel = { viewModel.cancelRemap() }
```
- **REASON**: real read/write/verify with typed confirmation, status, honest evidence labels, no decorative buttons.
- **IMPACT**: all Tuning buttons call real ViewModel operations; "SIMPAN BACAAN" is enabled only after a non-empty read; write requires typed `TULIS`; ECM reset requires typed confirmation; write cancellation during erase/write/finalize forces `recoveryRequired` and no automatic identify.
- **TEST**: `RemapEngineTest`, `RemapCodecTest`, `RemapChecksumTest`, `RemapSeedKeyTest`, `RemapVendorStrategiesTest`, `RemapExecutionGuardTest`, `RemapWorkflowTest`.
- **GIT DIFF**: new engine/guard/state files; `TuningScreen.kt` large rebuild.

### 5.2 FTDI permission/discovery/interface/endpoints/config/timeouts/errors
- **FILES**: `core/ftdi/FtdiSerialDriver.kt` (existing, verified), `core/usb/AdapterConnectionPolicy.kt`, `MainViewModel` permission flow, `MainActivity` USB receiver.
- **ACTUAL OLD CODE**: unchanged baseline driver, but permission denial and open failure were not surfaced per-protocol.
- **REPLACEMENT CODE**:
```kotlin
if (!usbManager.hasPermission(device)) { updateTimelineStage(USB_PERMISSION_GRANTED, false, ...); return false }
val conn = usbManager.openDevice(device) ?: return false
if (!conn.claimInterface(iface, true)) { conn.close(); ...; return false }
// bulk IN/OUT endpoints located; missing endpoint => close()
override suspend fun write(...) = conn.bulkTransfer(outEp, data, data.size, timeoutMs)
override suspend fun read(...) = conn.bulkTransfer(inEp, rawBuffer, rawBuffer.size, timeoutMs)
```
```kotlin
protocolManager.onAdapterOpenFailed("transport.open() gagal untuk ${device.deviceName}")
```
- **REASON**: tasks 5/6 require permission, discovery, open failure, timeout and resource cleanup to be real and visible.
- **IMPACT**: permission request is routed through `ACTION_USB_PERMISSION`; open failure marks `UNAVAILABLE`; `close()` releases the interface/connection; read/write use explicit timeouts.
- **TEST**: `AdapterConnectionPolicyTest`, `KlineTransceiverTest`, `BaselineProtocolTest`.
- **GIT DIFF**: driver unchanged in this task except interaction via protocol manager; `HondaProtocolEngine.kt` +911/−11.

---

## TASK 6 — Protocol Manager + USB stability

### 6.1 New protocol registry (single wire owner)
- **FILE**: `core/protocol/ProtocolManager.kt` (new)
- **ACTUAL OLD CODE**: none.
- **REPLACEMENT CODE**:
```kotlin
enum class ManagedProtocol(val labelEn: String, val labelId: String) {
    USB_LINK, ECU_IDENTIFICATION, LIVE_DATA, DTC, KEYLESS_ID, REMAP;
    val supportsExclusiveLease get() = this == KEYLESS_ID || this == REMAP
}
enum class ProtocolSessionStatus { UNAVAILABLE, IDLE, READY, ACTIVE, SUSPENDED, FAILED, RECOVERING }
fun onAdapterOpened(description: String)
fun onAdapterLost(reason: String)
fun onLiveDataStarted(detail: String)   // ignored while an exclusive lease is held
fun acquireExclusive(protocol: ManagedProtocol, detail: String): ProtocolLeaseResult
fun releaseExclusive(protocol: ManagedProtocol, success: Boolean, detail: String)
```
- **REASON**: per-protocol status, one owner of the wire, truthful transitions from observed events.
- **IMPACT**: statuses only change on real events; adapter close/loss revokes leases and downgrades sessions; live data cannot become `ACTIVE` while Keyless/Remap holds a lease.
- **TEST**: `ProtocolManagerTest` (7 tests: open → IDLE/READY, open failure, single-owner lease, close/loss revoke, live suspended under lease, identify failure → FAILED, ECU reset → ECU sessions IDLE with adapter open).
- **GIT DIFF**: new file + `HondaProtocolEngine` integration.

### 6.2 Engine gate + liveness before every transaction
- **FILE**: `HondaProtocolEngine.kt`
- **ACTUAL OLD CODE**: operations used `driver` without checking whether the USB device was still attached.
- **REPLACEMENT CODE**:
```kotlin
private fun requireLiveTransport(operation: String): UsbTransport? {
    val transport = driver
    val device = attachedDevice
    if (transport != null && transport.isOpen && device != null &&
        usbManager.deviceList.containsKey(device.deviceId)) return transport
    disconnectInternal("Perangkat USB hilang sebelum $operation.")
    return null
}
suspend fun verifyTransportHealth(): Boolean = withContext(Dispatchers.IO) { ... }
```
- **REASON**: "Jangan menganggap perangkat masih aktif hanya karena objek koneksi belum dibersihkan."
- **IMPACT**: every transaction (identify, live, DTC, keyless, remap, maintenance) requires a live transport; periodic health check (foreground monitor) closes a stale link and shows a real `FAULT`-style message.
- **TEST**: `ProtocolManagerTest`, `BaselineProtocolTest`, compile.
- **GIT DIFF**: `HondaProtocolEngine.kt` +911/−11.

### 6.3 Transport self-degradation and restore after finish/fail/cancel
- **FILES**: `UsbSerialLibraryTransport.kt`; `MainViewModel.kt` (`restoreLinkAfterKeyless/Remap`, cancel/finally paths)
- **ACTUAL OLD CODE**:
```kotlin
// I/O exception thrown to caller; transport still reported isOpen
// (no consecutive-failure accounting)
```
- **REPLACEMENT CODE**:
```kotlin
private fun noteIoFailure(reason: String) {
    consecutiveIoFailures++
    if (consecutiveIoFailures < MAX_CONSECUTIVE_IO_FAILURES || !isOpen) return
    isOpen = false
    CommunicationSniffer.logEvent(CommEventCategory.ERROR, CommDataStage.STATE_CHANGE,
        status = "TRANSPORT_DEGRADED", ...)
}
```
```kotlin
// read failure / verify failure / maintenance finally
restoreLinkAfterRemap(wasEcuAlive)
// write exception
if (!writeStarted) restoreLinkAfterRemap(wasEcuAlive)
```
- **REASON**: intermittent disconnect must not leave a dead port marked open; cancelled or failed Keyless/Remap must not leave Live Data suspended forever.
- **IMPACT**: after 3 consecutive I/O failures the transport marks itself closed; identify/live resume only when the ECU was alive before the operation; a started write never auto-restores (recovery requires read-back).
- **TEST**: `KlineTransceiverTest`, `ProtocolManagerTest`, `RemapEngineTest`; compile/tests pass.

### 6.4 Confirmed ECU reset invalidates stale ECU sessions
- **FILES**: `ProtocolManager.kt`, `HondaProtocolEngine.kt`, `MainViewModel.kt`
- **ACTUAL OLD CODE**: after a successful `RESET ECM`, `isEcuAlive`/`isEcuIdentified` and Live Data status from before the reset stayed in place; only the live stream was suspended.
- **REPLACEMENT CODE**:
```kotlin
// ProtocolManager
fun onEcuSessionReset(detail: String) = synchronized(lock) {
    if (!adapterOpen) return@synchronized
    if (exclusiveOwner == null) {
        set(ManagedProtocol.ECU_IDENTIFICATION, ProtocolSessionStatus.IDLE, detail)
        set(ManagedProtocol.LIVE_DATA, ProtocolSessionStatus.IDLE, detail)
        set(ManagedProtocol.DTC, ProtocolSessionStatus.IDLE, detail)
    }
    lastTransition = "ECU_SESSION_RESET"
    publish()
}
```
```kotlin
// HondaProtocolEngine
fun noteEcuSessionReset() {
    _isEcuAlive.value = false
    _isEcuIdentified.value = false
    _connectedModel.value = null
    ...
    protocolManager.onEcuSessionReset("ECU di-reset; ...")
}
```
```kotlin
// MainViewModel, successful reset only
if (report.outcome == RemapOutcome.SUCCESS) protocolEngine.noteEcuSessionReset()
```
- **REASON**: "Jangan menganggap perangkat masih aktif hanya karena objek koneksi belum dibersihkan." A reset ECU restarts; previous identification/live/DTC evidence is no longer valid.
- **IMPACT**: adapter remains open, ECU sessions return to `IDLE`, top bar stops claiming `ECU ONLINE`, and the message instructs a new identify after the restart.
- **TEST**: `ProtocolManagerTest` — "ecu reset clears ecu sessions but keeps the adapter open" (7th test).

---

## TASK 7 — Keep screen on (all menus, lifecycle-aware)

- **FILE**: `MainActivity.kt`
- **ACTUAL OLD CODE**:
```kotlin
override fun onResume() {
    super.onResume()
    viewModel.onAppResumed()
    viewModel.refreshStorage()
}
```
- **REPLACEMENT CODE**:
```kotlin
override fun onResume() {
    super.onResume()
    window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
    viewModel.onAppForegrounded()
    viewModel.onAppResumed()
    viewModel.refreshStorage()
}

override fun onPause() {
    window.clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
    viewModel.onAppBackgrounded()
    super.onPause()
}
```
- **REASON**: keep the screen on in every menu, release it when the app is not visible.
- **IMPACT**: flag is tied to the Activity lifecycle, not to a single screen; backgrounding also stops K-Line polling while keeping the port open.
- **TEST**: compile; lifecycle behaviour requires device confirmation (owner validation pending).

---

## TASK 8 — Startup gate 10 → confirmation → 2 s loading

- **FILES**: `MainActivity.kt` (render derived only from process-scoped `startupState`), `MainViewModel.kt`
- **ACTUAL OLD CODE**: loading screen could be entered through more than one path and the minimum-loading window was not enforced.
- **REPLACEMENT CODE**:
```kotlin
private const val STARTUP_LOADING_MIN_MS = 2000L
...
if (elapsed < STARTUP_LOADING_MIN_MS) delay(STARTUP_LOADING_MIN_MS - elapsed)
```
```kotlin
when (val state = startupState) {
    is StartupState.Idle -> LoadingScreen(0f)
    is StartupState.Examining -> IntegrityExaminationScreen(state.phases)
    is StartupState.AwaitingAcknowledgement -> {
        IntegrityExaminationScreen(...)
        IntegritySuccessDialog(
            onContinue = viewModel::acknowledgeIntegrity,
            writesInstallRecord = !state.report.debugBuild
        )
    }
    is StartupState.Starting -> LoadingScreen(state.progress)
    is StartupState.AwaitingTerms -> TermsScreen(...)
    is StartupState.Failed -> IntegrityFailureScreen(...)
    is StartupState.Ready -> MainAppScaffold(...)
}
```
```kotlin
// Debug builds now show the same confirmation window; no trust record is written.
pendingCommit = null
_startupState.value = StartupState.AwaitingAcknowledgement(outcome.report)
```
- **REASON**: mandatory visible 10-gate exam, confirmation window, and 2 s loading on every start with no bypass or double loading; a debug build no longer skips the window.
- **IMPACT**: single render branch per state; `startStartup()` is idempotent; the dialog text is truthful for debug ("no installation record written"); failure is terminal. Trust-record/in-process reuse paths keep their existing policy behaviour (no new FULL).
- **TEST**: `GateStagingTest`, `IntegrityGateTest`, `InstallationMarkerTest`, `TrustRecordTest`; compile and full suite pass.

---

## TASK 9 — DTC layout (Scan top / list middle / Hapus bottom)

- **FILE**: `DiagnosticsScreen.kt`
- **ACTUAL OLD CODE**: Scan and Clear were in one top Row; the list was not the scroll body between them.
- **REPLACEMENT CODE**:
```kotlin
// Task 9: Scan is the primary action and sits at the top of the menu.
...
// Bottom: Hapus DTC stays reachable after any scroll.
...
Text(t("HAPUS DTC", "DELETE DTC"), ...)
```
- **REASON**: fixed order required by task 9.
- **IMPACT**: Scan card at top; DTC rows in a scrollable middle `LazyColumn`; Hapus DTC at the bottom; clear remains `SUCCESS_VERIFIED` only when the ECU confirms.
- **TEST**: compile; `DiagnosticsScreen` has no unit test (UI layout), owner device validation pending.

---

## TASK 10 — Identity audit and renames

Detailed findings and retained external identifiers: see `IDENTITY_AUDIT.md` in this directory.

Summary of code changes:
- **FILES**: renamed package `com.revcore.android.data.flashpro` → `com.revcore.android.data.maps` (23 source + 9 test files, via `git mv` + import rewrite), `app/proguard-rules.pro` keep rule updated.
- **ACTUAL OLD CODE**: `package com.revcore.android.data.flashpro`, `class FlashProProfiles`, `data class FlashProProfile`, `enum class FlashProViewType`, `class FlashProDefinitionBuilder`, ids `flashpro.dat17`, labels `(FLASHPRO dat17)`.
- **REPLACEMENT CODE**: `package com.revcore.android.data.maps`, `object RevCoreProfiles`, `data class RevCoreProfile`, `enum class RevCoreViewType`, `class RevCoreDefinitionBuilder`, ids `revcore.dat17`, labels `(revcore.dat17)`.
- **REASON**: internal identities must be RevCore; auto-replacement was not used blindly — each occurrence was checked for asset/protocol/attribution context.
- **IMPACT**: no external protocol identifier, asset filename, or attribution was removed; asset paths `flashpro_profiles.json` / `flashpro_maps/` and the signed inventory list remain unchanged.
- **TEST**: full compile, 435 unit tests, `assembleDebug` after the rename; APK dex verified to contain `com/revcore/android/data/maps` and no `data/flashpro`.
- **GIT DIFF**: 32+ files renamed/modified; see `git status` / `git diff --stat`.

---

## Build verification commands (executed)

```text
gradle --offline -q :app:compileDebugKotlin          -> exit 0
gradle --offline -q :app:testDebugUnitTest           -> exit 0 (51 suites, 435 tests, 0 failures, 0 errors)
gradle --offline -q :app:assembleDebug               -> exit 0
```
Debug artifact (not an official release, not published):
`app/build/outputs/apk/debug/app-debug.apk` — 57,965,567 bytes — SHA-256 `5feb79affcaed28177defda5733df11813b9f9a7721d391de5a4432130582ab2`.
