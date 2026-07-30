# cmets-os changelog

Generated from `releases.jsonl`, GrapheneOS `releases.atom`, and per-fork `CHANGELOG.d/`.
Do not hand-edit; regenerate with `forks-release-manifest.sh`.

## About cmets-os

[cmets-os](https://github.com/cmets-os) is a GrapheneOS-based OS fork.

- Upstream project: [GrapheneOS](https://grapheneos.org/)
- Upstream source: [platform_manifest](https://github.com/GrapheneOS/platform_manifest)
- Full upstream release notes: [releases](https://grapheneos.org/releases) / [releases.atom](https://grapheneos.org/releases.atom)

cmets-specific notes appear under the “cmets” heading in later entries. Upstream GrapheneOS notes for each synced release are embedded under “Upstream (GrapheneOS …)” in `releases.atom`.

## 2026073002

- **tag:** `cmets-2026073002`
- **kind:** `local`
- **upstream_base:** `2026072900`

### cmets

_No CHANGELOG.d notes for this version._

## 2026073001

- **tag:** `cmets-2026073001`
- **kind:** `local`
- **upstream_base:** `2026072900`

### cmets

_No CHANGELOG.d notes for this version._

## 2026073000

- **tag:** `cmets-2026073000`
- **kind:** `local`
- **upstream_base:** `2026072900`

### cmets

_No CHANGELOG.d notes for this version._

## 2026072901

- **tag:** `cmets-2026072901`
- **kind:** `upstream`
- **upstream_tag:** `2026072900`

### Upstream (GrapheneOS 2026072900)

_See GrapheneOS release notes for `2026072900` (embedded in `releases.atom`)._

## 2026072900

- **tag:** `cmets-2026072900`
- **kind:** `local`
- **upstream_base:** `2026071500`

### cmets

_No CHANGELOG.d notes for this version._

## 2026072600

- **tag:** `cmets-2026072600`
- **kind:** `local`
- **upstream_base:** `2026071500`

### cmets

- Hide Users: hide existing full secondary users from the user list, shade, and lock screen while keeping Multiple users, Add user/Guest, and newly created users visible; recover via Dialer secret codes
- Shared encrypted storage: opt-in Shared folder encrypted with a dedicated TEE-wrapped Shared_CE key (unlocked after owner CE), visible as `/storage/emulated/<userId>/Shared` for enabled profiles; owner can wipe Shared from Settings
- Play Integrity spoof: per-app props/keybox/telephony spoof with GMS/Vending auto-include, user-imported hot-updatable store under `/data/misc/gms_attest_cfg` (no guaranteed DEVICE/STRONG)
- ADB data wipe: when armed, authorizing a new ADB host (USB confirm / allowDebugging / Wi-Fi pairing) triggers a full userdata wipe; charging and trusted-key reconnect do not; disable via Dialer code (default `*#8331#`)
- Dialer least privilege: Hide Users / ADB data wipe secret-code recovery uses narrow system APIs; Dialer no longer needs `WRITE_SECURE_SETTINGS` / `MANAGE_USERS` / `CREATE_USERS` for these features
- All Apps search no longer shows locked Private Space apps (fixes greyed “ghost” results and title leak when Private Space is locked or hidden)
- System Updater: use https://os.cmets.ai/releases/ with certificate pins for Cloudflare Universal (GTS WE1 + Root R4, ISRG X1/X2)
- SoftAP: when the vendor HAL returns an empty 5/6 GHz hotspot channel list, fill channels from Linux wireless-regdb for the real country code (no country-code spoof)
- Play Integrity attestation resign: when PI spoof is active, keystore2 re-signs the KeyMint attestation leaf with the imported (or ROM-seeded AOSP software) keybox private key and attaches the keybox chain; missing keybox or resign failure keeps the stock chain (props-only fallback). Soft default does not promise DEVICE/STRONG/GPay.
- Play Integrity spoof: skip expired keybox signing certificates and fall back to the other algorithm (e.g. RSA when AOSP soft ECDSA intermediate is past notAfter).

## 2026071500

- **tag:** `cmets-2026071500`
- **kind:** `upstream`
- **upstream_tag:** `2026071500`

### Upstream (GrapheneOS 2026071500)

_See GrapheneOS release notes for `2026071500` (embedded in `releases.atom`)._
