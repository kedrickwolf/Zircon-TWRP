### TWRP device tree for Redmi Note 13 Pro+ (zircon)

=========================================

The Redmi Note 13 Pro+ (codenamed _"zircon"_) is a high-end, mid-range smartphone from Xiaomi.
![Redmi Note 13 Pro+](https://cdn.cnbj0.fds.api.mi-img.com/b2c-shopapi-pms/pms_1695211669.22727628.png)

## Device specifications

Basic   | Spec Sheet
-------:|:-------------------------
CPU     | 2 x 2.8GHz Cortex-A715 + 6 x Cortex-A510
Chipset | Mediatek Dimensity 7200Ultra (mt6886)
GPU     | Mali-G610 MC6
Memory  | 12/16 GB RAM
Shipped Android Version | 13
Storage | 256/512 GB
Battery | Li-Po 5000 mAh, non-removable
Display | 1220 x 2712 pixels, 6.67 inches, 60/120/144 hz

## ⚠️ Device warning

This tree is for the **Redmi Note 13 Pro+ 5G (codename `zircon`, model `23090RA98C`)** only.
It is **NOT** for the regular Redmi Note 13 Pro 5G (different codename / partition table).
Flashing this on the wrong device can soft-brick or fail to boot.

## Files included in this tree

| File | Location | Purpose |
|---|---|---|
| `prebuilt/dtb` | `prebuilt/dtb` | Device tree blob (used in vendor_boot build) |
| `prebuilt/vbmeta_stock.img` | `prebuilt/vbmeta_stock.img` | Stock vbmeta from your extracted ROM — flash with `--disable-verity --disable-verification` |
| `prebuilt/vbmeta_system_stock.img` | `prebuilt/vbmeta_system_stock.img` | Stock vbmeta_system from your extracted ROM |
| `prebuilt/vbmeta_vendor_stock.img` | `prebuilt/vbmeta_vendor_stock.img` | Stock vbmeta_vendor from your extracted ROM |

**Stock images from your extracted ROM** (keep as reference/backup, do NOT flash unless restoring):
- `Extracted Official Zircon ROM/images/boot.img` — stock kernel (TWRP uses this kernel via the `boot` partition)
- `Extracted Official Zircon ROM/images/vendor_boot.img` — stock vendor_boot (restore if TWRP fails)

## Prerequisites

Before flashing TWRP you must:

1. **Unlock the bootloader** (official Xiaomi Mi Unlock). Unlocking **wipes all data**.
2. **USB debugging enabled** and `fastboot` working on your PC.
3. **Stock boot partition intact** — this tree uses `TARGET_NO_KERNEL := true`, meaning TWRP boots using your device's **stock kernel** from the `boot` partition, combined with the TWRP ramdisk in `vendor_boot`. Do not replace `boot.img` unless you know what you're doing.

## Step 1 — Disable AVB / verified boot (required)

The bootloader enforces verified boot (`BOARD_AVB_ENABLE := true`). Flashing a non-stock `vendor_boot` will be rejected unless AVB is disabled. Use the **stock vbmeta images from your own extracted ROM** (already copied into `prebuilt/`):

```bash
# From the TWRP tree directory:
fastboot --disable-verity --disable-verification flash vbmeta prebuilt/vbmeta_stock.img
fastboot --disable-verity --disable-verification flash vbmeta_system prebuilt/vbmeta_system_stock.img
fastboot --disable-verity --disable-verification flash vbmeta_vendor prebuilt/vbmeta_vendor_stock.img
```

> ⚠️ Flash only the vbmeta images that exist in your ROM. If `vbmeta_system` or `vbmeta_vendor` flashing fails, skip those — flash only `vbmeta`.

## Step 2 — Build TWRP

```bash
. build/envsetup.sh
lunch twrp_zircon-eng
make vendorbootimage -j$(nproc --all)
```

Output: `out/target/product/zircon/vendor_boot.img`

## Step 3 — Flash TWRP vendor_boot

```bash
fastboot flash vendor_boot out/target/product/zircon/vendor_boot.img
```

## Step 4 — Boot into TWRP

```bash
fastboot reboot recovery
```

If `fastboot reboot recovery` doesn't work, use your device's key combination (Volume Up + Power while connected) to enter recovery mode.

## To restore stock recovery (if needed)

```bash
fastboot flash vendor_boot "Extracted Official Zircon ROM/images/vendor_boot.img"
```

## To use TWRP (once booted)

- `adb sideload <zip>` — install zip via ADB (works even if `/data` is not decrypted)
- Internal storage access requires `/data` to decrypt (FBE metadata encryption). The tree enables `TW_INCLUDE_CRYPTO` + `TW_CRYPTO_USE_METADATA` — test that `/data` mounts before relying on backups/restore.

## Kernel note

`TARGET_NO_KERNEL := true` is set by design. This tree does **not** build a kernel `Image`. TWRP boots using your stock kernel from the `boot` partition (found in `Extracted Official Zircon ROM/images/boot.img`). There is no separate kernel to download — using a random kernel from the internet on mt6886 can hard-brick your device.

## Donate me

![mmqmem](https://img2.imgtp.com/2024/03/05/FdGtaESF.jpg)
