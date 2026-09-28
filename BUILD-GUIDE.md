# BUILD GUIDE — AOSP Android 16 (r4) for Sony Tama + Kernel 5.4

Repos live in the **Tama-5-4-lineage** org (forks of `Sony-SDM845-5-4` +
`bartcubbins`). Build target: **AOSP `android-16.0.0_r4`** (like SODP), not Lineage.

## 1. Server
- Ubuntu 22.04/24.04 x86_64
- Disk: **>= 250 GB**; RAM: **>= 16 GB** (32 GB recommended); CPU: >= 8 cores
- `repo`, `git`, AOSP build dependencies (see AOSP docs / `build-tools`)

## 2. Get the source
```sh
mkdir -p ~/aosp && cd ~/aosp
repo init -u https://android.googlesource.com/platform/manifest -b android-16.0.0_r4
git clone --depth 1 https://github.com/Tama-5-4-lineage/local_manifests .repo/local_manifests
repo sync -c --no-clone-bundle --force-sync -j$(nproc)
```

## 3. Build (kernel from source)
The kernel is built from source (`common-kernel/KernelConfig.mk` has
`BUILD_KERNEL := true`); the kernel 5.4 tree carries `AndroidKernel.mk`.
```sh
source build/envsetup.sh
lunch aosp_h8266-userdebug      # h8266 = XZ2 Dual; h8216 = XZ2
mka bacon                       # or: m otapackage
```

## 4. Output & flash
- A/B OTA zip: `out/target/product/akari/aosp_h8266-ota-*.zip` -> `adb sideload`
- Images: `boot.img`, `dtbo.img`, `system.img`, `vendor.img`, `product.img`, `vbmeta.img`
```sh
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot flash boot boot.img
fastboot flash dtbo dtbo.img
fastboot reboot
```
> `fastboot boot` is not supported on Sony -> always `fastboot flash`.

## 5. Dynamic partitions (retrofit)
The device tree (`device-sony-tama` / `device-sony-common`) is patched for
**retrofit dynamic partitions**: super config (system/vendor/oem), logical fstab,
`androidboot.dynamic_partitions[_retrofit]=true`, super_block_device sepolicy.
Kernel 5.4 is patched (initramfs skip + force_normal_boot + DT fstab) and `dm-linear`
is built-in.

## 6. Notes
- Port is experimental; expect bugs (display/camera/audio).
- Boot image uses SODP addresses (`tags=0x01E00000`, `ramdisk=0x02000000`).
- Software binaries: Tama has no 5.4 binaries; use the SODP Tama **v3** (4.19)
  odp blobs (`vendor-sony-tama`).
