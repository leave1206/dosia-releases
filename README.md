# DOSIA Releases

This repository hosts compiled release assets (`.dmg`, `.zip`) for **DOSIA**, a macOS desktop application.

The application source code is private and is not published here. This repo exists solely so the in-app updater can fetch release metadata and binaries without requiring users to authenticate.

## Downloads

Visit [Releases](https://github.com/leave1206/dosia-releases/releases) and pick the `.dmg` for your Mac's architecture (Apple Silicon → `-arm64.dmg`, Intel → non-arm64).

## Install on other Macs

If macOS Gatekeeper reports "file is damaged", run on the target machine:

```sh
xattr -cr /Applications/DOSIA.app
```

## Questions

Issues about the app itself should be reported through in-app channels (not here).

## plugin-compat.json — Plugin × DOSIA compatibility table

`plugin-compat.json` is consumed by DOSIA's in-app upgrade daemon to decide
whether a freshly-published plugin CLI version is safe to offer to users
running a particular DOSIA version. The daemon fetches this file every
24 hours (with a 6-hour client-side cache) and runs `resolveCompatibleUpgrade`
on each entry against the user's installed CLI version + current DOSIA app
version.

### Schema

```json
{
  "<plugin-name>": {
    "<cli-version>": {
      "minDosiaVersion": "x.y.z",
      "reason": "(optional) why this version requires DOSIA upgrade",
      "deprecated": false
    }
  }
}
```

- `<plugin-name>` matches the directory name under `plugins/` in DOSIA
  (e.g. `feishu`, `wechat`).
- `<cli-version>` is the npm package version of the plugin's CLI dep
  (e.g. `@larksuite/cli@1.0.22` → key `"1.0.22"`).
- `minDosiaVersion` — DOSIA app versions ≥ this can safely install this
  CLI version. Use the lowest DOSIA version where it works.
- `reason` — only shown to users when their DOSIA is older. Explain WHY:
  "新沙盒策略所需" / "loader schema v3" etc.
- `deprecated: true` — daemon stops offering this version (use to retract
  a release that turned out broken without rolling back the npm publish).

### How to add a new CLI version

1. CLI gets a new release on npm.
2. Test it against current DOSIA — does it need any DOSIA-side change
   (new IPC, new sandbox capability, schema bump)?
3. Open a PR adding one entry:
   - **No DOSIA change needed (99% of releases)**: copy the previous
     entry's `minDosiaVersion`. Users see the upgrade in their daemon
     within 6h.
   - **DOSIA change needed (1%)**: ship the DOSIA dmg first; in the same
     wave, set the new entry's `minDosiaVersion` = the new dmg's version.
     Users on older DOSIA see an amber "需先升级 DOSIA" hint instead of
     attempting an incompatible install.

### Failure modes

- **CLI version on npm but absent from this file**: the daemon refuses to
  offer it (fail-closed). This is intentional — every release should be
  documented here. If a user reports "I see v1.0.30 on npm but DOSIA isn't
  prompting me", check whether this file lists v1.0.30 yet.
- **Network failure fetching this file**: daemon falls back to its
  6-hour stale cache; further failure → falls back to the legacy
  `npm outdated` path which respects the in-dmg semver range. Users on
  the legacy path won't see cross-major upgrades but won't be broken.
