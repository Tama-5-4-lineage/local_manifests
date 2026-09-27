# BUILD GUIDE — LineageOS 23 (Android 16) untuk Sony Tama + Kernel 5.4

Panduan build di server. Semua repo di org **Tama-5-4-lineage** (fork dari
`Sony-SDM845-5-4` + `bartcubbins`).

## 1. Spesifikasi server
- OS: Ubuntu 22.04/24.04 x86_64
- Disk: **≥ 250 GB** (source + out + ccache)
- RAM: **≥ 16 GB** (disarankan 32 GB); swap ≥ 16 GB jika RAM pas-pasan
- CPU: ≥ 8 core
- `repo` (dari Google), `git`, `git-lfs` (opsional)

### Dependensi (Ubuntu)
```sh
sudo apt update
sudo apt install -y bc bison build-essential ccache curl flex g++-multilib gcc-multilib \
  git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev libelf-dev \
  liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils \
  lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev python3 python3-pip \
  openjdk-11-jdk-headless
```

## 2. Ambil source
```sh
mkdir -p ~/lineage && cd ~/lineage

# AOSP/Lineage manifest
repo init -u https://github.com/LineageOS/android -b lineage-23.0

# manifest org kita (arahkan kernel/device/vendor ke fork kita)
git clone --depth 1 https://github.com/Tama-5-4-lineage/local_manifests .repo/local_manifests

# sync (pertama kali lama)
repo sync -c --no-clone-bundle --force-sync -j$(nproc)
```

## 3. Build
```sh
source build/envsetup.sh

# Produk AOSP (device tree asli):
lunch aosp_h8266-userdebug          # XZ2 Dual; pakai aosp_h8216 untuk XZ2 single

# Produk Lineage (sudah ditambah di device tree kita):
# lunch lineage_akari-userdebug

mka bacon                            # hasil OTA
# atau:
# m otapackage                       # zip OTA
# m                                  # images (.img) saja
```

## 4. Output & flash
- A/B OTA: `out/target/product/akari/aosp_h8266-ota-*.zip` (atau `lineage_akari-ota-*.zip`)
  → `adb sideload` dari recovery.
- Images: `boot.img`, `dtbo.img`, `system.img`, `vendor.img`, `product.img`, `vbmeta.img`, dst.

Flash manual (bootloader unlocked):
```sh
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot flash boot boot.img
fastboot flash dtbo dtbo.img
# dst.
fastboot reboot
```
> **`fastboot boot` TIDAK didukung Sony.** Selalu `fastboot flash`.

## 5. Kernel
- **Dari source**: repo `Tama-5-4-lineage/kernel_sony_sdm845-5.4` (cubbins,
  `LA.UM.9.16.r1-18200-MANNAR.QSSI15.0`) + techpack + wlan.
- Build kernel lewat device tree (`AndroidKernel.mk` / `TARGET_KERNEL_VERSION`) atau
  script `common-kernel/build-kernels-clang.sh`.
- `kernel-sony-msm-5.4-common` juga menyediakan **prebuilt** `kernel-dtb-*` + `dtbo-*.img`.
  Untuk build source, pastikan mekanisme kernel memakai source (bukan prebuilt).

## 6. Software binaries (odm) — opsional
Untuk 5.4 Tama, unduh dari `https://opendevices.sony.net/aosp-on-xperia-open-devices/downloads/software-binaries`
(AOSP Android 16 / Kernel 5.4 / Tama), lalu flash ke partisi `oem`/`odm` sesuai
kebutuhan device tree.

## 7. Catatan & pitfall
- Ini porting komunitas yang **eksperimental** — ekspektasikan bug (display/camera/audio).
- Alamat boot image 5.4 (SODP): `tags=0x01E00000`, `ramdisk=0x02000000`.
- Device tree android-nya (b-mr1) AOSP-oriented; `lineage_akari.mk` kita sekadar
  membungkusnya jadi produk Lineage — tuning Lineage lebih lanjut mungkin diperlukan.
- Simpan `restore/boot.img` + `dtbo.img` asli untuk pemulihan.
