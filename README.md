# local_manifests — Tama-5-4-lineage

Manifest untuk membangun **Android 16 (LineageOS 23)** di Sony Tama (SDM845:
akari / XZ2, akatsuki, apollo) dengan **kernel 5.4** dari `cubbins`
(`LA.UM.9.16.r1-18200-MANNAR.QSSI15.0`).

Semua repo kernel/device/vendor diarahkan ke fork org kita **`Tama-5-4-lineage`**
(remote `cubbins` dan `fork`), sisanya tetap dari `sonyxperiadev` / `NXP`.

## Repo yang di-fork di org ini
- Kernel 5.4: `kernel_sony_sdm845-5.4`, `..._techpack_{audio,camera,display,dataipa,datarmnet,datarmnet-ext}`,
  `..._wlan_{fw-api,qca-wifi-host-cmn,qcacld-3.0}`
- Kernel common/headers: `kernel-sony-msm-5.4-common`, `kernel-sony-msm-5.4-headers`, `qti-headers`
- Device: `device-sony-{common,tama,akari,akatsuki,apollo,sepolicy}`
- Vendor: `vendor-sony-tama`

## Build

```sh
# 1. Source Lineage (Android 16)
repo init -u https://github.com/LineageOS/android -b lineage-23.0

# 2. Ambil local_manifests org kita
git clone --depth 1 https://github.com/Tama-5-4-lineage/local_manifests .repo/local_manifests

# 3. Sync
repo sync -c --no-clone-bundle --force-sync -j8

# 4. Build
source build/envsetup.sh
lunch aosp_h8266-userdebug      # produk device tree (h8266 = XZ2 Dual; h8216 = XZ2)
mka bacon                       # atau: m otapackage
```

## Catatan
- **Kernel dari source** (`kernel/sony/msm-5.4/kernel` ← fork cubbins). `common-kernel`
  juga menyediakan prebuilt; untuk build dari source pastikan mekanisme kernel
  (`AndroidKernel.mk` / `build_shared.sh`) memakai source, bukan `kernel-dtb-*` prebuilt.
- Default device tree memakai produk **`aosp_h8266` / `aosp_h8216`**. Untuk produk
  Lineage, tambahkan `device/sony/akari/lineage_akari.mk` (inherit `aosp_h8266.mk` +
  `vendor/lineage/config/common_full_phone.mk`).
- Boot image 5.4 memakai alamat SODP (`tags=0x01E00000`, `ramdisk=0x02000000`) — lihat
  `device-sony-tama/PlatformConfig.mk`.
- `fastboot boot` tidak didukung Sony → selalu `fastboot flash`.
- Software binaries (odm) untuk 5.4 disediakan terpisah (opendevices.sony.net) — pakai
  yang cocok Android 16 / Kernel 5.4 Tama.
