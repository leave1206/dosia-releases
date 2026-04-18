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
