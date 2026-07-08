# GWRT OpenWrt

GWRT OpenWrt release and upgrade metadata for CS-02/IPQ60xx builds.

This repository is the main public project for full firmware releases,
release notes, and cross-version upgrade policy.

Small dashboard/theme/config hotfixes are published through:

```text
https://github.com/rothdren-lion/gwrt-ota
```

## Release Channels

```text
channels/stable/re-cs-02.json
```

The stable channel manifest describes the latest release, compatible device,
full firmware assets, and any patch OTA path that is safe for the current build.

## Upgrade Rules

- Small UI/theme/script/ACL/config-default changes: patch OTA.
- Kernel, driver, DTS, partition, libc, package ABI, firewall base, or storage layout changes: full sysupgrade/factory.
- Patch OTA is never used across incompatible base firmware.
- Full firmware is never auto-flashed by the dashboard; it only opens the normal LuCI flash page.

See [docs/upgrade-design.md](docs/upgrade-design.md).
