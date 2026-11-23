# REVCORE — PROJECT IDENTITY

**Owner:** Otoy Sullivan
**Public credit / developer identity:** `otoy`
**Application:** RevCore
**Package:** `com.revcore.android`
**Version:** `1.0` (`versionCode 1`) — frozen unless the owner orders a change
**Platform:** Android, Kotlin + Jetpack Compose
**Philosophy:** **“RevCore — Jembatan untuk belajar, bukan pagar untuk membatasi
pengetahuan.”**

---

## 1. What the name means

RevCore is the owner's diagnostic and learning application for Honda PGM-FI
motorcycles. It is intended to make ECU communication, Live Data, DTC, MAP structure,
Keyless identity, and remap mechanics understandable and inspectable. It is not a
“tuning magic box”, not a clone of a commercial product, and not a platform for
unverified write operations.

The visible identity is consistently **RevCore**:

- app label: `RevCore` (`res/values/strings.xml`, `res/values-in/strings.xml`)
- package and namespace: `com.revcore.android`
- official certificate subject: `CN=RevCore Official, OU=Release, O=RevCore, L=Jakarta,
  ST=DKI Jakarta, C=ID`
- official certificate SHA-256:
  `c649cb4fd7e67686a99c1df31936a489b89e62a877b8d6b358f64d027a18d098`
- owner credit shown in the app: `otoy`

The current `README.md` in the baseline is still the original AI Studio/Android Studio
boilerplate. It does **not** describe RevCore and must not be treated as the project
identity. This document and `REVCORE_PROJECT_RULES.md` are authoritative.

---

## 2. What may never be rebranded

Without an explicit written order from Otoy, a successor AI must not:

- rename the app, package, namespace, launcher label, or certificate identity;
- replace the credit `otoy` or claim authorship;
- change the philosophy text;
- replace the theme or visual identity;
- remove features to “simplify” the project;
- change `versionName`, `versionCode`, `applicationId`, or signing configuration;
- generate a new signing key or certificate;
- rewrite external protocol/hardware identifiers for cosmetic consistency.

The identity audit intentionally preserved external technical names. Examples:

- `FTDI`, `FT232R`, `K-Line`, `Keihin`, `Shindengen`, `Honda PGM-FI` — required for
  technical accuracy and user context;
- `revcore_profiles.json`, `revcore_maps/manifest.tsv` — signed asset names; renaming
  breaks Gate 10 and the asset inventory;
- research paths and attribution comments such as `VomConfig`, `BinLoader`,
  `flashproDomain.*` — provenance of the reverse-engineered schema;
- `HDiag` in one KDoc statement — a factual statement that the current Live Data
  policy is *not* derived from HDiag.

Internal code names, log tags, UI labels, tests, and the package
`com.revcore.android.data.maps` were migrated to RevCore. The full old/new table is in
`02_CURRENT_WORKING_STATE/project/delivery/rebuild_161126_20261122/IDENTITY_AUDIT.md`.

---

## 3. Owner and public history

- Owner and final decision-maker: **Otoy Sullivan**.
- The public developer repository is `https://github.com/otoy6661/Revcore-Project`.
- Release `v0.3.0` (17 November 2026) published a **developer `:core` library**, not the
  full application source. It deliberately excluded UI, security/Gate 10, remap/flash,
  real ECM database, real MAP set, and proprietary Live Data catalogs.
- The owner separately decided to publish the full official APK `RevCore_161126.apk` as
  a release asset. That was an owner decision about the owner's own work and data.
- The repository/project is **not** an OSI open-source project. Usage follows
  `NOTICE.md`: “RevCore is free to inspire, not free to impersonate. Learn from it.
  Build your own. Be proud of your work. Respect the original.”

Publication history is recorded for context only. **This migration backup does not grant
permission to publish anything.** The current order forbids commit, push, release, and
publication without a new explicit owner instruction.

---

## 4. Product boundaries

RevCore's honesty rules are part of its identity:

- no fake sensor values, no default fill for missing ECU data, no auto-calibration;
- estimates are labelled as estimates (“Prakiraan AFR” / “Estimated AFR”), never as
  measured AFR;
- no success popup before a real, verified outcome;
- no write/erase command unless the protocol path and recovery are evidenced;
- build success is not feature success; unit tests are not ECU validation;
- debug builds are never presented as verified official builds;
- no claim of physical vehicle testing without a recorded vehicle test.

These boundaries are not “nice-to-have” styling. They are owner requirements and must
survive migration.
