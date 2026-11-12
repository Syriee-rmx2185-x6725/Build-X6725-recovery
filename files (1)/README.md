# Infinix Smart 10 (X6725) recovery tree - UNTESTED DRAFT

Derived from your stock vendor_boot_a.img + boot_a.img. Facts found:
- SoC platform ums9230 (Unisoc), hardware string ums9230_X6725_go, A/B + Virtual A/B, dynamic partitions
- boot_a: header v4, NO ramdisk, uncompressed Image, kernel 5.15.194-android13 (GKI)
- vendor_boot_a: header v4, one 34 MB lz4 ramdisk that already contains the stock recovery (/system/bin/recovery, fastbootd, minadbd, recovery.fstab) + first_stage_ramdisk + 176 recovery modules
- Userdata: f2fs, FBE aes-256-xts v2 + inlinecrypt_optimized, metadata encryption

Build: push this folder to a GitHub repo, run Actions -> "Build X6725 recovery". Output: vendor_boot.img.
Flash (keep stock backup first!):
  fastboot flash vendor_boot_a vendor_boot.img
  fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img   (use a stock vbmeta.img)
Then boot recovery with Vol Up + Power or: adb reboot recovery

Items to verify (couldn't from the dumps): super partition size, backlight sysfs path, touchscreen input name,
DTB extraction (prebuilt/dtb is the raw vendor_boot dtb blob), tags offset, Android 14.1 TWRP branch compatibility
with a 5.15 kernel/vendor API 33 (may need twrp-12.1 branch instead).

## Verified against reference OrangeFox 14.1 image (OrangeFox-Unofficial_X6725.img)
- vendor_boot ramdisk layout: entry 0 = platform ramdisk, entry 1 = recovery ramdisk (type 2). Matches BOARD_INCLUDE_RECOVERY_RAMDISK_IN_VENDOR_BOOT.
- Build is Android 14 (SDK 34, AP2A.240905.003): PLATFORM_VERSION := 14.
- Product name in reference: fox_X6725. Lunch target: fox_X6725-eng.
- recovery.fstab replaced with the reference's /data line (fileencryption=aes-256-xts:aes-256-cts:v2+inlinecrypt_optimized, keydirectory, reservedsize).
- Device init file init.recovery.ums9230_X6725_go.rc copied from the reference.
- Kernel/modules still from your stock dumps (reference kernel not extracted).
