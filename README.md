# ImmortalWrt MT798x Build (openwrt-24.10)

Based on `padavanonly/immortalwrt-mt798x-6.6` branch `openwrt-24.10-6.6`.

## Device Variants

| Variant | Platform |
|---|---|
| mt7975-ipailna-high-power | mt7986 |
| mt7975-ipailna-high-power_dae | mt7986 |
| mt7981-ax3000 | mt7981 |
| mt7981-ax3000-dae | mt7981 |
| mt7986-ax4200-bpir3_mini | mt7986 |
| mt7986-ax6000 | mt7986 |
| mt7986-ax6000_dae | mt7986 |

## Build

```bash
git clone --depth=1 -b openwrt-24.10-6.6 https://github.com/padavanonly/immortalwrt-mt798x-6.6.git openwrt
cd openwrt
bash ../01_prepare.sh
bash ../build_all.sh
```

## Packages

Edit `package.conf` to add or remove extra packages.

The GitHub Actions workflow builds every upstream 24.10 defconfig variant, enables xz squashfs compression, packages the images with high-compression 7z settings, and publishes the archive to GitHub Releases.
