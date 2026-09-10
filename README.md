# OrangeFox recovery for Redmi Note 14 4G (tanzanite)

This repository contains an OrangeFox `fox_12.1` build setup and the
Tanzanite recovery device tree. The target is the Redmi Note 14 4G with the
MediaTek MT6789 (Helio G99 Ultra), Android 14/HyperOS firmware, Android boot
header v4, dynamic partitions, and A/B with virtual A/B updates.

Recovery is stored in the `vendor_boot` partition. The checked-in
`stock/vendor_boot.img` is a 64 MiB reference image from the target firmware;
the workflow verifies its SHA-256 before using it to preserve the stock
vendor ramdisk layout. Do not substitute an image from another region,
firmware release, or device variant.

## Build locally

Use a supported Ubuntu host with the Android build prerequisites, Git LFS, and
the `repo` tool installed:

```bash
ROOT=$(pwd)
git lfs pull
git clone https://gitlab.com/OrangeFox/sync.git OrangeFox_sync
chmod +x OrangeFox_sync/orangefox_sync.sh
OrangeFox_sync/orangefox_sync.sh --branch fox_12.1 --path "$ROOT/fox_12.1"
rsync -a --delete device/xiaomi/tanzanite/ \
  fox_12.1/device/xiaomi/tanzanite/

cd fox_12.1
export FOX_AB_DEVICE=1
export FOX_VIRTUAL_AB_DEVICE=1
export FOX_VENDOR_BOOT_RECOVERY=1
export FOX_INSTALLER_VENDOR_BOOT_RAMDISK_INSTALL=1
export FOX_REFERENCE_VENDOR_BOOT_IMAGE="$ROOT/stock/vendor_boot.img"
export FOX_TARGET_DEVICES=tanzanite
source build/envsetup.sh
lunch twrp_tanzanite-eng
mka vendorbootimage -j"$(nproc --all)"
```

The expected recovery output is a generated
`vendor_ramdisk_recovery.cpio` (or its temporary `.lz4` form) under
`fox_12.1/out/target/product/tanzanite/`. The workflow validates that the
ramdisk contains `system/bin/recovery`, `system/bin/fastbootd`,
`sbin/foxstart.sh`, and `system/etc/recovery.fstab`. Any
`vendor_boot.img` produced by the Android packaging target is treated only as
an additional diagnostic and is rejected if it is byte-identical to the stock
reference.

## GitHub Actions

Run **Build OrangeFox Tanzanite** with **workflow_dispatch**. The job verifies
the LFS reference image, syncs OrangeFox `fox_12.1`, builds the checked-in
device tree, and uploads `OrangeFox-tanzanite-recovery-ramdisk.zip` together
with the individual artifacts.

## Flashing and hardware status

This tree is configured to produce a vendor_boot image, but a successful
compile is not proof that recovery boots or that decryption, touch, display,
USB, or fastbootd work on every regional variant. Unlock the bootloader and
keep a complete matching stock firmware package before any hardware testing.
The workflow intentionally produces and verifies the recovery ramdisk first;
it does not claim that a tested, directly flashable vendor_boot image exists.
