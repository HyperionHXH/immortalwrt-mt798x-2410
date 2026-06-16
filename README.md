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

Manual `workflow_dispatch` runs can build either all variants or one selected variant. Push and pull-request runs still build the full matrix in parallel. The workflow also caches `openwrt/dl` so repeated runs can reuse downloaded source archives where possible.

## CI Notes

- 2026-06-16: `logs_74264526223` showed six variants completed package/release steps. The missing variant was `mt7975-ipailna-high-power`, whose log stopped during `make download` without a compiler error. The workflow keeps full matrix parallelism for speed and retries downloads with lower per-job parallelism to reduce download-source instability.
- Lines that contain `No squashfs images found under openwrt/bin/targets/mediatek` in successful logs are just GitHub Actions echoing the shell script. Treat them as failures only if they appear after the script actually runs and the step exits non-zero.
- 2026-06-17: Added `openwrt/dl` caching and a manual single-variant selector for rerunning one failed firmware without rerunning the whole matrix.
