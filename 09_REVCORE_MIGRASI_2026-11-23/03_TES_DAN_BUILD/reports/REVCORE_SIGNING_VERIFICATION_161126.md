# REVCORE — SIGNING VERIFICATION REPORT

Artifact: `dist/RevCore build 161126.apk`
Verification time: 2026-11-16, after export
Report ID: `REVCORE-SIGNING-VERIFY-161126`
Method: independent commands run after the build script finished, against the exported copy.

---

## 1. SIGNATURE VERIFICATION

Tool: Android build-tools 36.0.0 `apksigner.jar`, run with the Android Studio JBR.

```
Verifies
Verified using v1 scheme (JAR signing): false
Verified using v2 scheme (APK Signature Scheme v2): true
Verified using v3 scheme (APK Signature Scheme v3): true
Verified using v3.1 scheme (APK Signature Scheme v3.1): false
Verified using v4 scheme (APK Signature Scheme v4): false
Number of signers: 1
```

v1 is intentionally disabled: minSdk is 26, and the manifest injection step re-signs with
`--v1-signing-enabled false --v2-signing-enabled true --v3-signing-enabled true` so the injected
`assets/ui_integrity_manifest.json` is covered by a modern scheme.

---

## 2. CERTIFICATE

| Field | Value |
|---|---|
| DN | `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta, ST=DKI Jakarta, C=ID` |
| SHA-256 | `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098` |
| SHA-1 | `388637719c378c3c4b9823c7784c093989b9bd6f` |
| Key algorithm / size | RSA / 2048 |
| Public key SHA-256 | `debd3e8ed682223b4f1cd169cbf78a1e2bc07d8834ed01508a38e7c4f4f44b29` |
| Expected established fingerprint | `c649cb4f…18d098` |
| Match | **YES** |
| Debug certificate (`CN=Android Debug`) | absent |

The certificate fingerprint equals the fingerprint already recorded for the previous official
build (151126) and the pin inside the signed policy, so this is the same official signing identity,
not a replacement key.

---

## 3. SIGNED CONTENT BINDING

| Check | Result |
|---|---|
| `assets/integrity_policy.json` present and signed | Yes, `SHA256withRSA` |
| Policy `signerCertSha256` equals APK certificate | Yes |
| Policy `expectedSignerSha256` equals APK certificate | Yes |
| Policy package / schema / debug flag | `com.revcore.android` / 1 / false |
| Policy asset inventory | 165 assets, 3 protected, no extra, no missing |
| Protected asset hashes | All match the policy |
| Native library inventory | Matches the policy |
| `assets/ui_integrity_manifest.json` present and signed | Yes, `SHA256withRSA` |
| UI manifest signer equals APK certificate | Yes |
| UI manifest bound to packaged policy digest | Yes |
| UI resource / DEX / structure digests | Internally consistent, entry hashes match |
| Policy bytes unchanged after manifest injection and re-sign | Yes |

---

## 4. ARTIFACT INTEGRITY

| Check | Result |
|---|---|
| File exists, non-zero size | Yes, 39,689,642 bytes |
| SHA-256 (exported) | `b2b6c5bd68cc9c727c2babe0c510b928a6135fa7ef5e9d8847275887c8647432` |
| Checksum file | `dist/RevCore build 161126.sha256.txt`, identical hash |
| Exported copy equals built `app-release.apk` | Yes, identical SHA-256 |
| ZIP/archive integrity (`unzip -t`) | No errors |
| Manifest parse (`aapt2 dump badging` / `xmltree`) | Succeeds |
| Package | `com.revcore.android` |
| versionCode / versionName | `1` / `1.0` |
| `debuggable` | false (attribute absent) |
| Key material packaged inside APK | none found |

---

## 5. POSITIVE/NEGATIVE EVIDENCE

- The **new** Sentinel strings are present in `classes.dex` (titles, `GERBANG`, `LANJUTKAN`,
  `KELUAR`, all seven anomaly headings).
- The **previous** artifact `RevCore build 151126.apk` does not contain them.
- The exported APK is therefore a new signed artifact containing this task's implementation, not a
  renamed older file.

---

## 6. LIMITS OF THIS VERIFICATION

This report verifies the artifact's signature, certificate, packaging, and signed-content binding.
It does not verify runtime behavior on a device, Keystore operations, UI rendering, or any ECU
function. Those remain the checklist items in
`REVCORE_OFFICIAL_RELEASE_BUILD_REPORT_161126.md` §7.

No password, keystore content, or private key appears in this report.
