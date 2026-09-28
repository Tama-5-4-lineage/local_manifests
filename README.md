# local_manifests — Tama-5-4-lineage (AOSP)

Local manifests for building **AOSP Android 16 (r4)** on Sony Tama (SDM845:
akari / XZ2, akatsuki, apollo) with **kernel 5.4** from `cubbins`
(`LA.UM.9.16.r1-18200-MANNAR.QSSI15.0`).

All kernel/device/vendor projects point to the **Tama-5-4-lineage** org
(remote `cubbins` and `fork`); the rest comes from `sonyxperiadev` / `NXP`.

## Repos in this org
- Kernel 5.4: `kernel_sony_sdm845-5.4`, `..._techpack_{audio,camera,display,dataipa,datarmnet,datarmnet-ext}`,
  `..._wlan_{fw-api,qca-wifi-host-cmn,qcacld-3.0}`
- Kernel common/headers: `kernel-sony-msm-5.4-common`, `kernel-sony-msm-5.4-headers`, `qti-headers`
- Device: `device-sony-{common,tama,akari,akatsuki,apollo,sepolicy}`
- Vendor: `vendor-sony-tama` (SODP Tama v3 binaries/odm)

## Build
See `BUILD-GUIDE.md`. Short version:
```sh
repo init -u https://android.googlesource.com/platform/manifest -b android-16.0.0_r4
git clone --depth 1 https://github.com/Tama-5-4-lineage/local_manifests .repo/local_manifests
repo sync -c --force-sync -j$(nproc)
source build/envsetup.sh && lunch aosp_h8266-userdebug && mka bacon
```

## Notes
- Kernel is built **from source** (`kernel-sony-msm-5.4-common/KernelConfig.mk`
  has `BUILD_KERNEL := true`).
- Device tree patched for **retrofit dynamic partitions**.
- `fastboot boot` is not supported on Sony -> use `fastboot flash`.
