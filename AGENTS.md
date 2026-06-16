# AGENTS.md - ImmortalWrt MT798x 24.10 Build

## Project

Build ImmortalWrt 24.10 firmware for MT798x devices using GitHub Actions.

Upstream source:

- Repository: `padavanonly/immortalwrt-mt798x-6.6`
- Branch: `openwrt-24.10-6.6`

## Build Variants

Use the upstream 24.10 defconfigs exactly as the source tree provides them:

- `mt7975-ipailna-high-power` -> `mt7986`
- `mt7975-ipailna-high-power_dae` -> `mt7986`
- `mt7981-ax3000` -> `mt7981`
- `mt7981-ax3000-dae` -> `mt7981`
- `mt7986-ax4200-bpir3_mini` -> `mt7986`
- `mt7986-ax6000` -> `mt7986`
- `mt7986-ax6000_dae` -> `mt7986`

Do not copy the 23.05 `mt7986-ax4200` or `mt7986-ax6000-256m` variants into this project; they are not present in the 24.10 upstream defconfig directory.

## Wi-Fi Notes

The 24.10 defconfigs already use `luci-app-mtwifi-cfg` and do not enable `wifi-profile`.

Keep the standard `/sbin/wifi` path intact. Do not apply the old `luci-app-mtk + wifi-profile` workaround from older 21.02/23.05 experiments unless a build log proves the 24.10 upstream changed.

## Release Compression

The workflow enables:

- `CONFIG_TARGET_SQUASHFS_XZ=y`
- 7z LZMA2 archive compression with `-md=256m -mfb=273 -ms=on -myx=9`

This keeps the firmware itself smaller and then packages release artifacts as tightly as practical in GitHub Actions.

Do not enable `luci-app-passwall_INCLUDE_Shadowsocks_Rust_Client` or `luci-app-passwall_INCLUDE_tuic_client` in CI unless the runner has substantially more disk space. On GitHub-hosted `ubuntu-22.04`, those options build Rust/LLVM from source and have failed with `No space left on device`.
