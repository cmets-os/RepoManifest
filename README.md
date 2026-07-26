# cmets-os platform_manifest

> **Disclaimer.** cmets-os is developed primarily for **internal use at cmets**. This public repository is provided **as-is, at your own risk**. The changes described here have **not been systematically tested**, and **ongoing support is not guaranteed**. cmets-os is **not affiliated with** GrapheneOS or Google. Several deltas below are deliberate divergences from GrapheneOS policy and are **not intended for upstream**.

## What this is

**cmets-os** is a GrapheneOS-based Android OS fork (AOSP plus GrapheneOS hardening) maintained for cmets operational needs. It tracks GrapheneOS on Pixel-oriented trees via `cmets-*` integration branches, keeps that security baseline, and adds pragmatic UX / ops features—including ones GrapheneOS deliberately does not ship.

This repository is the `repo` manifest overlay: a vendored GrapheneOS snapshot plus cmets-os fork pins. See [Quick start](#quick-start) below for sync instructions.

## Why it exists

- Close a Private Space search leak when lock / “hide when locked” already exist.
- Offer practical UI hiding of secondary users against casual inspection or coercion—not forensic deniability.
- Provide an opt-in Shared encrypted folder so hard profile isolation does not push people into unsafe transfer workarounds.
- Allow per-app Play Integrity / region spoof for selected apps only (accepted risk; not “certify the whole OS”).
- Wipe userdata on **new** ADB host authorization, alongside GrapheneOS Duress (no credible silent ADB pairing story).
- Restore SoftAP 5/6 GHz channel lists from wireless-regdb for the device’s real country code—without spoofing the country.

## How it differs from GrapheneOS / AOSP

For each item: upstream position, our stance, what we ship.

### 1. Private Space search “ghosts”

**Upstream.** AOSP Private Space and Android Help expect locked private apps to stay off launcher search / OS surfaces; launchers must honor lock and hide-when-locked. Stock Pixel Launcher largely does; AOSP Launcher3 has leaked quiet / hidden private apps in All Apps search (also reported on other AOSP-based ROMs).

**Our stance.** With hide-when-locked already a product feature, search results that still show badged private apps are a bug / leak, not intended behavior.

**What we ship.** Launcher3 filters private quiet / hidden apps out of All Apps search so presence of those apps is not disclosed that way.

### 2. Hide Users

**Upstream.** GrapheneOS improves multi-user (higher limits, logout → at rest, etc.) but rejects “hidden / deniable” profiles as a real security property: profile metadata remains detectable (including via ADB); true deniability would need a much heavier design (e.g. VM-based). See [os-issue-tracker#5247](https://github.com/GrapheneOS/os-issue-tracker/issues/5247).

**Our stance.** Useful against casual glance and low-skill coercion even when forensics can still find profiles—like not putting a neon sign on a safe. We do **not** claim plausible deniability under forensic analysis.

**What we ship.** Device-wide Hide Users: snapshot-hide existing secondary users from lists / shade / lock screen; self-hiding Settings entry; Dialer secret-code escape hatches. **Deliberate divergence** from GrapheneOS; not for upstream.

### 3. Shared encrypted storage

**Upstream.** AOSP isolates each user’s primary external storage; cross-space sharing is meant to go through controlled UI (Sharesheet / Photo Picker when Private Space is unlocked) or off-device paths. GrapheneOS keeps per-profile CE isolation (and Storage Scopes); it does not add a shared CE directory between profiles.

**Our stance.** Absolute isolation without a safe on-device path shifts leak risk onto the user (USB sticks, cloud messengers, shady “file share” apps). An opt-in, OS-mediated Shared volume with its own encryption key is preferable and adds no threat if unused.

**What we ship.** Opt-in Shared directory (`/storage/emulated/<userId>/Shared`) with a separate FBE/fscrypt key (CE-class), mountable for opted-in users / Private Space. **Deliberate divergence** for operational use of profiles.

### 4. Play Integrity / region spoof (v1)

**Upstream.** GrapheneOS treats Play Integrity device/strong gates as anti-competitive, not real security; the supported path is apps using Android hardware attestation and allowlisting GrapheneOS keys ([attestation compatibility guide](https://grapheneos.org/articles/attestation-compatibility-guide)). Project maintainers have stated that shipping spoofing at GrapheneOS scale is a losing game (fingerprint bans, keybox revocation). See [os-issue-tracker#1986](https://github.com/GrapheneOS/os-issue-tracker/issues/1986).

**Our stance.** Refusing the whole stack on purity grounds is not always acceptable for our ops. We accept a limited risk: spoof only for selected apps (in practice a narrow set), not a global “look certified” mode.

**What we ship.** Per-app Play Integrity prop imitation + optional user-imported keybox path, plus optional telephony/region spoof; GMS/Play auto-included while any client is spoofed. v1 substitutes attestation cert material; **DEVICE/STRONG are not guaranteed**. **Deliberate divergence**; not for upstream. Follow-up (attestation resign with keybox private key) is planned separately.

### 5. ADB data wipe

**Upstream.** GrapheneOS ships Duress PIN/password (full wipe), USB-C port controls, and hardened ADB authorization UX. It does not wipe on authorizing a new ADB host.

**Our stance.** Useful next to Duress. There is no legitimate scenario where a new ADB host is authorized without the user’s knowledge; charge-only and already-trusted keys must not wipe.

**What we ship.** When armed: wipe userdata on new-host ADB auth (USB confirm / wireless pairing); not on charge, not on trusted-key reconnect. Self-hiding Settings entry; Dialer secret-code disarm only.

### 6. SoftAP 5/6 GHz (wireless-regdb fallback)

**Upstream.** SoftAP bands follow country code and vendor HAL channel lists. GrapheneOS rejects a productized blanket regulatory bypass / `force-country-code` ([os-issue-tracker#2665](https://github.com/GrapheneOS/os-issue-tracker/issues/2665)); fixes that stay within local rules for a real country are a different matter. Empty SAP lists for some regions (e.g. where Pixels are not sold) are a known pain point.

**Our stance.** Missing SAP channels look like vendor/regdb gaps, not a reason to fake the country. Filling lists from Linux wireless-regdb for the **same** ISO stays within local rules.

**What we ship.** When the HAL SAP list is empty for 5/6 GHz but a real country code is known, SoftAP channel lists are filled from wireless-regdb for that ISO—**without** forcing or spoofing country code.

### What we do not claim

- Forensic plausible deniability for hidden users  
- Guaranteed Play Integrity `MEETS_DEVICE_INTEGRITY` / `MEETS_STRONG_INTEGRITY`  
- Fitness for public production use, compatibility guarantees, or ongoing support  
- Any endorsement by GrapheneOS of Play Integrity spoofing or Hide Users  

Follow-ups landed in forks (see workspace `Done/`): Dialer as a thin secret-code front-end (least privilege); Play Integrity attestation resign with imported/seeded keybox private key (AOSP soft default does not promise DEVICE/STRONG/GPay).

---

`default.xml` is a thin overlay: it includes a vendored GrapheneOS snapshot (`grapheneos.xml`) and replaces only forked projects with our tagged revisions.

> **Generated files:** `default.xml`, `grapheneos.xml`, `releases.atom`, and `CHANGELOG.md` are produced by workspace scripts. `releases.jsonl` is append-only from merge/capture. Do not hand-edit generated artifacts for a release.

## Quick start

```bash
mkdir cmets-os && cd cmets-os
repo init -u https://github.com/cmets-os/platform_manifest.git -b main -m default.xml
repo sync -j$(nproc)
```

Pin a release branch/tag of this manifest repo when you publish one (recommended over floating `main`).

## Layout

| File | Role |
|------|------|
| `default.xml` | Entry point for `repo init` — include upstream + cmets overrides (generated; committed) |
| `grapheneos.xml` | Vendored GrapheneOS `default.xml` snapshot at the upstream base (generated; committed) |
| `releases.jsonl` | Append-only version journal (upstream merges + local captures) |
| `releases.atom` | Generated Atom changelog (pretty-printed; GOS notes + cmets bullets; ends with static About entry; committed) |
| `CHANGELOG.md` | Short markdown mirror (About section first, then versions; generated; committed) |
| `LICENSE` | Apache-2.0 |

## Remotes

- **GrapheneOS / AOSP** — defined inside `grapheneos.xml` (unchanged from upstream).
- **cmets** — `https://github.com/cmets-os/` — used only for forked projects.

## Branching

Integration branch on every fork: **`cmets-<upstream-default>`** (e.g. `cmets-17`, `cmets-main`).

- Develop and merge **only** on `cmets-*`.
- Upstream GrapheneOS/AOSP updates are merged **into** `cmets-*` (never reset onto stock `17`).
- Recommended GitHub default branch for cmets-os forks: `cmets-17` (or `cmets-main`). Stock `17` can remain as a remote-tracking reference only.

Release tags: `cmets-<YYYYMMDDNN>` in one numeric series. Local captures prefer UTC `YYYYMMDD00`, then the next free NN via `cmets_next_free_version`. Upstream sync prefers the GrapheneOS id `T` as `cmets-T`, but if that number is already taken (e.g. by a local capture) or not greater than the current max, the script bumps to the next free `cmets-V`. The GrapheneOS tag is always recorded as `upstream_tag` in `releases.jsonl` (`version` may be `> T`).

## Fork naming

| Manifest / GitHub name | Tree path |
|------------------------|-----------|
| `platform_frameworks_base` | `frameworks/base` |
| `platform_packages_apps_Settings` | `packages/apps/Settings` |
| `platform_packages_apps_Dialer` | `packages/apps/Dialer` |
| `platform_packages_apps_Launcher3` | `packages/apps/Launcher3` |
| `platform_packages_apps_GmsCompat` | `packages/apps/GmsCompat` |
| `platform_packages_apps_Updater` | `packages/apps/Updater` |
| `platform_packages_modules_Wifi` | `packages/modules/Wifi` |
| `platform_system_sepolicy` | `system/sepolicy` |
| `platform_system_vold` | `system/vold` |
| `platform_system_security` | `system/security` (overrides AOSP `platform/system/security`) |

## Release workflow (parent workspace)

```bash
# 1) Merge every pending GrapheneOS tag in order (no skipping)
./Scripts/forks-merge-upstream.sh

# 2) Optional: commit user-facing notes in forks, then snapshot local work
#    Forks/<repo>/CHANGELOG.d/<version>.md  (see Scripts/templates/CHANGELOG.d/)
./Scripts/forks-capture-local.sh

# 3) Write Repo/default.xml + grapheneos.xml + releases.atom + CHANGELOG.md
./Scripts/forks-release-manifest.sh
# optional: --version YYYYMMDDNN  --push-tags
```

Steps 1–2 can run many times before 3. The journal records each upstream tag and each local capture; step 3 rebuilds the full Atom/markdown changelog from the journal.

Per-fork notes: `CHANGELOG.d/<YYYYMMDDNN>.md` on `cmets-*` (one bullet list; multi-repo features need only one file across forks). Upstream text comes from [GrapheneOS releases.atom](https://grapheneos.org/releases.atom) keyed by `upstream_tag`. The generated feed is pretty-printed and always ends with a static **About cmets-os** entry (links to GrapheneOS site, `platform_manifest`, and upstream changelog).

Full maintainer docs for the merge/capture/release scripts: [cmets-os/scripts](https://github.com/cmets-os/scripts) (README). In a local cmets-os workspace checkout, see also `CONTRIBUTING.md` / `CONTRIBUTING.RU.md` at the workspace root.

## Official builds

For production / `OFFICIAL_BUILD=true`, the Updater base URL must point at **your** server — never the official GrapheneOS releases URL. See GrapheneOS build docs and your Updater config.
