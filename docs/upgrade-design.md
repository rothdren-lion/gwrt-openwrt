# GWRT Cross-Version Upgrade Design

## Goals

1. Keep small updates fast with patch OTA.
2. Prevent unsafe patching across incompatible firmware bases.
3. Keep full sysupgrade/factory as the only path for major system changes.
4. Make update decisions deterministic from manifest metadata.

## Version Model

GWRT uses monotonically increasing build numbers:

```text
GWRT V20260606.7
GWRT V20260606.8
GWRT V20260606.9
```

The integer after the final dot is the build number. Every patch OTA or full
firmware release must increase it.

## Compatibility Fields

Each release should declare:

```json
{
  "device": "jdcloud,re-cs-02-gwrt2",
  "target": "qualcommax/ipq60xx",
  "base": "6.18.37-r0-0bad892",
  "rootfs_generation": 1,
  "build": 8
}
```

Patch OTA is only valid when `device`, `target`, `base`, and
`rootfs_generation` match the installed firmware.

## Update Types

### Patch OTA

Use patch OTA only for narrow file updates:

```text
www/luci-static/*
usr/share/ucode/luci/template/*
usr/share/luci/menu.d/*
usr/share/rpcd/acl.d/*
usr/libexec/*
etc/config/gwrt
etc/uci-defaults/*
etc/init.d/*
```

Patch OTA must include `sha256`, device/target checks, and an allowed build
range:

```json
{
  "type": "patch",
  "from_min_build": 7,
  "from_max_build": 7,
  "to_build": 8
}
```

The patch should be cumulative for the declared range. For example, if the
latest build is 10, publish one patch from 8-9 to 10 instead of requiring every
router to chain through 8 -> 9 -> 10.

### Full Sysupgrade

Use sysupgrade when any base system component changes:

```text
kernel
drivers
DTS/board files
partition layout
libc/toolchain ABI
firewall base behavior
network stack defaults
preinstalled package ABI or dependency graph
root filesystem layout
```

The dashboard should show an update prompt but must route users to LuCI's
normal firmware flash page. It should not silently run sysupgrade.

### Factory Image

Factory images are for first install, recovery, or vendor-to-GWRT conversion.
They are not an in-place OTA path.

## Cross-Version Decision

Router update logic should follow this order:

1. Read current `build`, `device`, `target`, `base`, and `rootfs_generation`.
2. Load the channel manifest.
3. If device/target mismatch: reject.
4. If a patch entry covers the current build and compatibility fields match:
   offer patch OTA.
5. Otherwise, if a sysupgrade image exists:
   show full firmware update and open LuCI flash page.
6. If only factory exists:
   show manual install/recovery guidance.

## Manifest Shape

```json
{
  "schema": 1,
  "channel": "stable",
  "device": "jdcloud,re-cs-02-gwrt2",
  "target": "qualcommax/ipq60xx",
  "latest": {
    "version": "GWRT V20260606.8",
    "build": 8,
    "base": "6.18.37-r0-0bad892",
    "rootfs_generation": 1
  },
  "upgrade": {
    "patch": {
      "from_min_build": 7,
      "from_max_build": 7,
      "url": "https://github.com/rothdren-lion/gwrt-ota/releases/download/gwrt-v20260606.8/gwrt-ota-8.tar.gz",
      "sha256": "74420f655921ed6a9bd7697000fc87ae7efa208a8ad48196a0a14804228e4e8e",
      "size": 96825,
      "reboot": false
    },
    "sysupgrade": null,
    "factory": null
  }
}
```

## Rollback

Patch OTA backs up overwritten files under:

```text
/etc/gwrt-ota/backups/YYYYMMDD-HHMMSS
```

Full firmware rollback should use normal sysupgrade with a known-good image.
