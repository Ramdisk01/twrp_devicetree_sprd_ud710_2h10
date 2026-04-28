
# TWRP Device Tree for UNISOC UD710 (Tiger T710)

Minimal TWRP device tree for UD710-based devices with eMMC/UFS. Includes prebuilt kernel, FBE decryption, and essential recovery components.
（You can find the eMMC and UFS versions of the tree from different branches.）


> **Note on Dynamic Partitions (Android 10+ / `super` layout)**  
> This repository currently **does not support dynamic partition devices**. If your target device uses a `super` partition with logical volumes for `system`/`vendor`/`product`, you need to adapt the device tree yourself.  
> For reference, see [https://github.com/alghiffaryfa19/device_tree_twrp_jingpad] for a working example on a dynamic‑partition device.
>
> The **`emmc` branch** contains vendor‑specific blobs/configs that have **not** been de‑customised – it is kept only as an **example** of how a real static‑partition device tree looks on eMMC storage.  
> When adapting the tree for your own device, always follow the general checklist in the rest of this README.


UD710 New Device Adaptation Checklist

When using this tree for another UD710 device, verify/modify the following:

1. Kernel & DTBO

· Replace prebuilt/Image.gz-dtb and prebuilt/dtbo.img with your device's kernel and dtbo.

2. Touch Firmware

· If your device requires touch firmware files (e.g., .bin), place them under recovery/root/vendor/firmware/ (though not necessarily this path). Ensure they are properly referenced.

3. Partition Layout

· Update recovery.fstab with correct block device paths (/dev/block/by-name/...) and filesystem types.
· Adjust partition sizes in BoardConfig.mk:
  · BOARD_{SYSTEM,VENDOR,PRODUCT,...}_IMAGE_PARTITION_SIZE
  · If using super partitions, set BOARD_SUPER_PARTITION_SIZE and group sizes.

4. Device-Specific Variables (BoardConfig.mk)

· TARGET_DEVICE – set to your device codename.
· TARGET_OTA_ASSERT_DEVICE – comma-separated list of devices this tree supports.
· TARGET_RECOVERY_PIXEL_FORMAT – usually RGBX_8888 or BGRA_8888; check your display.
· Display resolution: TW_THEME (e.g., watch_mdpi, portrait_hdpi), and optionally TW_DEVICE_WIDTH/HEIGHT in BoardConfig.mk.
· TW_ROTATION – set to 0, 90, 180, or 270 depending on screen orientation.

5. Binary Compatibility

· If your device runs a different Android version (e.g., 10/11), update the binaries in recovery/root/ (e.g., vold, gatekeeper, keymaster) with ones extracted from your device.
· Pay attention to library dependencies – replace .so files under recovery/root/vendor/lib64/ as needed.

6. SELinux

· After building, flash recovery and check if SELinux is permissive (getenforce in adb shell). If not, manually apply the patch in patches/permissive-selinux.patch:
  · Edit system/core/init/selinux.cpp in your source tree according to the patch.
  · Note: Adding androidboot.selinux=permissive to BOARD_KERNEL_CMDLINE is ineffective; Spreadtrum devices read cmdline only from dtb.

7. Ueventd

· Review recovery/root/ueventd.rc and ensure device nodes (e.g., touch, display) match your kernel’s sysfs paths. Add missing entries if needed.

8. Init Scripts

· If your device requires special services (e.g., for touch, keymaster), modify recovery/root/init.recovery.service.rc or add a device-specific .rc file.

Once all items are checked, follow the build instructions in the main README.


## Building

```bash
# Initialize TWRP manifest (twrp-9.0 branch)
repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni -b twrp-9.0
repo sync

# Setup environment and build
source build/envsetup.sh
lunch omni_ud710_2h10-eng
make recoveryimage -j$(nproc --all)
```

Output: out/target/product/ud710_2h10/recovery.img

Credits
· Atlas - providing FBE crypto solutions for the TWRP of Spreadtrum   -- https://github.com/atlas4381
