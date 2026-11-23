# Research archive index — Bahan Otopsi

## 1. The archive is preserved, not modified

| Item | Value |
|------|-------|
| Original | `C:\Users\Lord Otoy\Desktop\Bahan Otopsi.zip` |
| Backup copy | `06_RESEARCH_ARCHIVE/archive/Bahan Otopsi.zip` |
| Size | 122,456,873 bytes |
| SHA-256 | `b9db1554b41952874f2121059724410f4569ce073952c601424b5fc59ffae1b9` |
| Entries | 64 files |
| Verification | Original and copy hashes identical; copy comparison logged in `copy-archive.log` |

The original was read only. It was not renamed, moved, modified, re-zipped, or overwritten. Do not do any of those things. If an analysis needs extracted files, extract to a new temporary directory and leave the archive untouched.

Privacy: the archive contains third-party Windows binaries, firmware images, an APK, and `H-Diag/login.ini`. Treat the whole archive as private. Do not upload it to AI services, public repositories, cloud storage, or chat. Do not inspect or disclose credential-like contents.

## 2. Contents (top-level groups)

| Group | What it contains | Why it matters to RevCore |
|-------|------------------|---------------------------|
| `H-Diag/` | Windows PC diagnostic application (`HDiag.exe`), Qt 5 runtime DLLs, USB driver package (`libusb_device*.inf`, WinUSB co-installers), `login.ini`, QtQuick plugin files | A PC-side reference for Honda diagnostic behaviour and USB/libusb adapter handling. Used as comparative evidence, not as source code copied into RevCore. |
| `RevCore/` | `RevCore build 201126.apk` plus `.sha256.txt`, and `.official-release-*` status/log/console files | An intermediate RevCore build and its official-release log. Not the baseline. The baseline remains the 161126 APK in `01_BASELINE_161126/`. |
| `Flashpro/` | `firmware.bin`, `firmware.signed.flash`, `FLASHPRO [1.0].apk` | FlashPro adapter firmware/application artefacts used in the FlashPro/FTDI reconstruction research. Firmware bytes are evidence about the adapter, not a RevCore ECU write feature. |
| `RevCore Keyless ID log.txt` | A captured Keyless ID session log | Evidence for the Keyless request/response sequence. It does not by itself prove a working Keyless operation on a vehicle. |
| `ecu-diagnostic-remapping/` | Flasher/Scanner Windows tools by a third party ("FRENGKI"), `FTD2XX_NET.dll`, user guide PDF, README, disclaimer, screenshots | A third-party ECU diagnostic/remapping toolkit used for comparison. It is not RevCore code and must not be redistributed as part of RevCore. |

The full flat file list captured at backup time is in `archive-contents.txt`.

## 3. What this archive is evidence for

- The existence and behaviour of a PC-side Honda diagnostic tool (`H-Diag`) and its USB driver stack.
- The FlashPro adapter's firmware/application artefacts and the shape of its update format.
- A captured Keyless ID exchange and the command/response structure it suggests.
- A third-party diagnostic/remapping tool's UI and workflow for comparison.

Evidence labels apply: archive contents are `STATIC_STRING_ONLY` or `RECONSTRUCTED_FROM_BINARY` unless a test exercised them. An archive artefact is not `HARDWARE_VALIDATED`, and it is not proof that RevCore's implementation is correct on an ECU.

## 4. What this archive is not evidence for

- It is not proof that RevCore's Keyless read/renew/reset works on a motorcycle.
- It is not proof that RevCore can or should write an ECU.
- It is not a licence to copy third-party binaries, firmware, or proprietary data into RevCore or a public release.
- It does not replace the official build, signing, or Gate 10 evidence.

Do not turn a hypothesis from this archive into a fact. If a claim is needed, point to the specific file, offset, string, or report and label the evidence level.

## 5. Relationship to the rest of the backup

- `01_BASELINE_161126/` — the official baseline APK and source. Never overwrite with the 201126 APK from this archive.
- `02_CURRENT_WORKING_STATE/` — the current work tree, including the research/audit documents written during development.
- `07_PROJECT_HISTORY/` — chronology of how the archive's findings were used.
- `08_ARCHITECTURE_AND_FEATURES/` — implementation description, with evidence labels.
- `10_KNOWN_ISSUES/` — which research conclusions remain unverified or contradictory.

## 6. Handling rules for the successor AI

- Read `archive-contents.txt` first; do not unzip the whole archive unless a specific question needs it.
- If extraction is necessary, extract to a fresh temp folder, record which files were read, and delete the temp folder afterwards. Never extract into the repository or the backup tree.
- Never execute `HDiag.exe`, `FLASHER BY FRENGKI.exe`, `SCANNER BY FRENGKI.exe`, or any archive binary. Analysis is static only unless Otoy explicitly authorizes a sandboxed run.
- Never use `login.ini` contents or any credential-like data.
- Never modify, re-zip, or replace the archive. The hash above is the reference.
- Never publish the archive or its contents.
