# REVCORE — OWNER AUTHORITY AND RULES FOR THE SUCCESSOR AI

**Owner:** Otoy Sullivan
**Executor of this backup:** Kodir / Qoder
**Document date:** 2026-11-23
**Status:** rules in force; migration acceptance pending owner decision

---

## 1. Authority hierarchy

1. **Otoy** is the owner, final reviewer, and final acceptor of every change.
2. The AI is an analysis and implementation tool. It proposes, implements when ordered,
   and reports evidence. It does not decide product direction, identity, release, or
   acceptance.
3. An earlier approval does not generalize. Approval to build, sign, commit, publish, or
   change a policy applies only to the exact scope and moment approved.
4. If an instruction conflicts with an owner rule, stop and ask. Do not “interpret” a
   prohibition into permission.

The owner's recorded corrections are evidence of intent. Re-read them before making a
change that touches startup, Gate 10, signing, identity, or the existing feature set.

---

## 2. Absolute prohibitions

These remain in force unless Otoy explicitly revokes them in writing for a specific task.

### Safety and ECU

- Do not send write, erase, reset, or renew commands to an ECU without a real, evidenced
  protocol path and an explicit owner instruction.
- Do not fabricate protocols, checksums, seed/key algorithms, EEPROM addresses, ECU
  responses, or hardware test results.
- Do not turn a hypothesis into a fact in code, UI, reports, or comments.
- Do not use a debug APK as a substitute for an official Release APK.
- Do not claim physical ECU or vehicle validation that did not happen.

### Gate 10 and trust

- Do not delete, disable, weaken, or bypass Gate 10.
- Do not change gate names, algorithms, policy format, trust-record format, or the
  startup policy approved by Otoy.
- Do not remove a protected asset from the policy to make a build pass.
- Do not rename a failing gate to PASS or silently ignore a FAIL.
- Do not regenerate the trust record manually or manufacture an installation marker.
- Do not create a new bypass in debug or release.
- Do not edit the generated `integrity_policy.json` or `ui_integrity_manifest.json` by
  hand. Rebuild through the official script so both are regenerated and signed.

### Signing identity

- Do not ask for, read, store, display, log, or record the signing password.
- Do not change, rename, move, regenerate, or replace the official key files.
- Do not change the signing certificate or the pinned certificate SHA-256.
- Do not commit, upload, or package `.p12`, `.jks`, `[berkas kunci lokal - tidak dipublikasikan]`, or any secret.
- Otoy enters the password locally in the existing masked dialog. The AI must not ask him
  to paste it into chat.

### Repository and release

Without explicit owner approval:

- no `git commit`, `git push`, force-push, branch change, or history rewrite;
- no deletion of branches, commits, tags, or reflog evidence;
- no remote change;
- no GitHub release, ZIP distribution, upload, or publication;
- no version, `applicationId`, package, label, or signing change;
- no source “cleanup” that changes behavior;
- no reformatting of the entire tree just to make it look tidy.

### Secrets and the migration package

- Do not place passwords, private keys, keystore contents, or decrypted secret material
  in reports, prompts, source, logs, or this package.
- Do not send secret files to an AI service or external tool.
- The private backup lives outside this package. Treat its path as sensitive but its
  contents as off-limits.

---

## 3. Source and evidence discipline

Before changing code:

1. Read `00_READ_FIRST/AI_MIGRATION_START_HERE.md` and the relevant architecture file.
2. Inspect the actual source at the current path; do not rely on this backup's summary
   alone.
3. Identify whether the change touches Gate 10, signing, assets, identity, protocol,
   storage, or UI policy. If yes, read the corresponding rule and report.
4. Verify the current state with `git status`, source reads, tests, and (where possible)
   a build. Do not assume the backup snapshot is still current if the owner has edited
   the tree after 2026-11-23.

Evidence labels to use in reports:

- **VERIFIED** — observed by an executed command/test against the actual artifact.
- **IMPLEMENTED** — code exists and is wired, but not yet validated on hardware.
- **BUILD SUCCESSFUL** — compilation/assembly passed; says nothing about feature behavior.
- **UNIT TESTED** — JVM test passed; says nothing about a real device or ECU.
- **SIMULATED / RECONSTRUCTED** — behavior inferred from binaries/firmware, tested with
  synthetic or recorded frames.
- **HARDWARE NOT TESTED** — no physical adapter/ECU/vehicle run.
- **OWNER VALIDATION PENDING** — Otoy has not accepted the result.

Never use IMPLEMENTED as a synonym for VERIFIED. Never use “works” when only compilation
passed.

---

## 4. Required change report format

For every source change, report in this order:

1. **FILE** — exact path.
2. **LOKASI** — function/line region.
3. **ACTUAL OLD CODE** — the real code being replaced, copied from the tree.
4. **REPLACEMENT CODE** — the real new code, not a sketch.
5. **REASON** — why the change is needed, tied to an owner requirement or evidence.
6. **IMPACT** — affected behavior, assets, gates, signing, tests, or compatibility.
7. **TEST** — commands and actual results; distinguish unit/build/device/ECU.
8. **GIT DIFF** — the actual diff, plus a list of new/changed/deleted/renamed files.

Do not use omission placeholders such as “rest of methods unchanged” in the code blocks.
If a report is long, split it by file but keep the actual code.

A completion report must state:

- what was implemented and what was only inspected;
- what tests ran and their counts;
- what remains untested or blocked;
- whether any prohibited action was taken (the answer must be “none”);
- that owner acceptance is still pending.

---

## 5. Communication and workflow

- Otoy's working language is Indonesian; technical identifiers stay in English.
- Do not flood him with repeated summaries of information already in the package.
- Do not create new work orders, prompts, or scope expansions unless there is a concrete
  need and he approves.
- Do not stop reverse-engineering merely because something is unknown. Implement the
  evidenced part, label the experimental part honestly, and state the gap.
- Do not ask the owner to run terminal commands that the AI can run itself. Report only a
  real blocker (missing credential, absent hardware, permission denial).
- When a task is ordered to delivery, execute it to delivery. Do not stop for
  re-approval at every internal step; stop only at a genuine owner decision or safety
  boundary.

---

## 6. Boundary of this migration

This backup is a snapshot for continuity, not a transfer of authority. It does not:

- authorize a new release;
- change the signing identity;
- approve the uncommitted working tree;
- accept the current APK as hardware-validated;
- permit publication of source, keys, research data, or APKs.

**Owner decision and migration approval remain with Otoy.**
