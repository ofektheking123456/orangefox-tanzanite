# OrangeFox recovery for Redmi Note 14 4G (tanzanite)

This repository contains an OrangeFox `fox_12.1` build setup and the
Tanzanite recovery device tree. The target is the Redmi Note 14 4G with the
MediaTek MT6789 (Helio G99 Ultra), HyperOS 3.1 Android 16 Global firmware
`OS3.0.302.0.WOGMIXM`, Android boot header v4, dynamic partitions, and A/B
with virtual A/B updates.

`fox_12.1` is the OrangeFox source/manifest branch name, not the Android
version of the target firmware. The current OrangeFox sync tool does not
provide an Android 16/`fox_16.1` branch; the Android 16 target is represented
by the stock vendor_boot reference and the Tanzanite device tree.

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
OrangeFox_sync/orangefox_sync.sh --branch 12.1 --path "$ROOT/fox_12.1"
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

The expected recovery outputs are a generated `vendor_boot.img` and its vendor
ramdisk (`vendor_ramdisk.cpio.lz4`, copied to `vendor_ramdisk_recovery.cpio`)
under `fox_12.1/out/target/product/tanzanite/`. The workflow validates that the
ramdisk contains `system/bin/recovery`, `system/bin/fastbootd`,
`sbin/foxstart.sh`, and `system/etc/recovery.fstab`. Any
`vendor_boot.img` produced by the Android packaging target is required,
validated as a vendor_boot v4 image, copied to the artifact as
`orangefox-vendor_boot.img`, and rejected if it is byte-identical to the stock
reference.

## GitHub Actions

Run **Build OrangeFox Tanzanite** with **workflow_dispatch**. The job verifies
the LFS reference image, syncs OrangeFox branch `12.1` into `fox_12.1`, builds
the checked-in device tree, and uploads
`OrangeFox-tanzanite-recovery-ramdisk.zip` together with the generated
`orangefox-vendor_boot.img`, ramdisk, and stock reference artifacts.

## Flashing and hardware status

This tree is configured to produce an OrangeFox vendor_boot image, but a
successful compile is not proof that recovery boots or that decryption, touch,
display, USB, or fastbootd work on every regional variant. Unlock the bootloader
and keep a complete matching stock firmware package before any hardware testing.
The workflow now produces and verifies the OrangeFox vendor_boot image, but it
has not been hardware-tested on the device.
