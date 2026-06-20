# ImmortalWrt 24.10 固件自定义与编译指南

本项目用于通过 GitHub Actions 编译 MT798x 平台的 ImmortalWrt 24.10 固件。一般不需要在本地安装编译环境，只需修改插件清单并在 GitHub 网页上运行工作流。

## 一、准备自己的仓库

1. 打开本项目的 GitHub 页面，点击右上角 **Fork**，创建到自己的账号下。
2. 进入 Fork 后的仓库，打开 **Settings > Actions > General**。
3. 在 **Workflow permissions** 中选择 **Read and write permissions** 并保存。工作流需要该权限创建 Release 并上传固件。
4. 打开仓库的 **Actions** 页面。如果 GitHub 提示工作流尚未启用，点击启用。

项目使用以下源码：

- 源码仓库：`padavanonly/immortalwrt-mt798x-6.6`
- 源码分支：`openwrt-24.10-6.6`
- 工作流文件：`.github/workflows/mt798x.yml`
- 插件清单：`package.conf`

## 二、增加插件

编辑仓库根目录的 `package.conf`，每行填写一个软件包名称。例如：

```text
luci-app-ddns
luci-app-openclash
luci-app-passwall
luci-app-ttyd
```

填写时注意：

- 只写软件包名称，不要写 `CONFIG_PACKAGE_`，也不要在末尾写 `=y`。
- 每行只能放一个包名，行首和行尾不要加空格。
- 空行会被忽略。
- 注释行必须以 `#` 开头。
- LuCI 插件通常以 `luci-app-` 开头，但不是所有软件包都有网页管理界面。
- 插件依赖会由 ImmortalWrt 配置系统自动选中，因此增加一个插件后，固件体积可能增加很多。

Passwall 的可选组件也通过 `package.conf` 添加，例如：

```text
luci-app-passwall
luci-app-passwall_INCLUDE_Xray
luci-app-passwall_INCLUDE_Hysteria
luci-app-passwall_INCLUDE_SingBox
```

不要随意添加 Rust 版 Shadowsocks、TUIC 等大型组件。它们可能明显增加编译时间和磁盘占用，之前的 CI 曾因 Rust 主机工具链接时磁盘不足而失败。

## 三、减少插件

如果插件是通过 `package.conf` 添加的，删除对应行即可。例如不需要 OpenClash，就删除：

```text
luci-app-openclash
```

需要注意：

- `package.conf` 只负责额外增加软件包。
- 如果某个包已经被上游设备的 `defconfig` 默认选中，仅从 `package.conf` 删除它并不能强制关闭。
- 某个插件可能被其他插件作为依赖自动选中，这种情况下也不能通过删除一行去掉。
- 不建议删除 MTK Wi-Fi 驱动、`mtwifi-cfg`、基础网络、LuCI、内核或启动相关组件，否则固件可能无法联网、无法进入管理页面，甚至无法启动。

如果确实要删除上游默认组件，应先在本地源码中运行 `make menuconfig` 检查依赖关系，再修改对应设备的配置。没有确认依赖前，不要直接在工作流里强制写入 `# CONFIG_PACKAGE_xxx is not set`。

## 四、在哪里修改插件

### 方法 A：直接在 GitHub 网页修改

1. 打开仓库根目录的 `package.conf`。
2. 点击右上角铅笔图标。
3. 增加或删除软件包行。
4. 点击 **Commit changes**。

如果提交到 `main` 分支，会自动触发全部 7 个变种并行编译。

### 方法 B：在电脑上修改

```bash
git clone https://github.com/你的用户名/immortalwrt-mt798x-2410.git
cd immortalwrt-mt798x-2410
```

编辑 `package.conf` 后提交：

```bash
git add package.conf
git commit -m "update firmware packages"
git push
```

在 Git Bash 中路径应使用 `/`，不要用 Windows 的反斜杠 `\`。

## 五、使用 GitHub Actions 编译

### 方式一：提交到 main 后自动全量编译

向 `main` 分支 push 后，工作流 `mt798x_24_10_CI` 会自动开始，全部固件变种并行编译。

适合以下情况：

- 准备正式发布全部设备固件；
- 插件清单已经验证；
- 希望所有变种保持相同插件配置。

### 方式二：手动编译一个变种

1. 打开仓库的 **Actions** 页面。
2. 左侧选择 **mt798x_24_10_CI**。
3. 点击右侧 **Run workflow**。
4. 在 `build_variant` 中选择需要的设备变种。
5. 点击绿色 **Run workflow** 按钮。

选择 `all` 会编译全部变种；选择具体名称时，只有该变种会真正下载和编译，其他矩阵任务会立即跳过。

### 推荐的单型号测试流程

如果只想测试一次插件变化，不想提交后立刻全量编译，可以使用测试分支：

```bash
git switch -c test-packages
# 修改 package.conf
git add package.conf
git commit -m "test package changes"
git push -u origin test-packages
```

然后到 **Actions > mt798x_24_10_CI > Run workflow**：

1. 分支选择 `test-packages`；
2. `build_variant` 选择一个实际使用的设备；
3. 运行并确认固件正常；
4. 验证通过后再把改动合并到 `main`。

## 六、支持的固件变种

| 变种 | 平台 |
|---|---|
| `mt7981-ax3000` | MT7981 |
| `mt7981-ax3000-dae` | MT7981 |
| `mt7975-ipailna-high-power` | MT7986 |
| `mt7975-ipailna-high-power_dae` | MT7986 |
| `mt7986-ax4200-bpir3_mini` | MT7986 |
| `mt7986-ax6000` | MT7986 |
| `mt7986-ax6000_dae` | MT7986 |

名称带 `dae` 的变种使用上游对应的 dae 配置。不要只根据名称猜测硬件型号，应以自己的路由器型号、闪存布局和当前使用的上游配置为准。

## 七、查看编译进度和下载固件

### 查看进度

1. 打开仓库的 **Actions** 页面。
2. 点击最新的 `mt798x_24_10_CI` 运行记录。
3. 点击具体设备任务查看日志。

绿色对勾表示成功，红色叉号表示失败，灰色任务通常是手动单变种编译时被跳过的其他变种。

### 下载固件

编译成功后，工作流会为每个变种创建单独的 GitHub Release：

1. 回到仓库首页；
2. 点击右侧 **Releases**；
3. 找到名称包含所需变种和日期的 Release；
4. 下载 `.7z` 文件并解压。

压缩包里可能有多个 `squashfs` 镜像。必须根据设备型号、启动介质和当前刷机方式选择正确文件：

- `factory` 一般用于特定原厂系统首次刷入；
- `sysupgrade` 一般用于兼容固件内升级；
- 不同 NAND、eMMC、SD 卡或 NOR 布局的镜像不能混用。

刷写错误镜像可能导致设备无法启动。刷机前应保存原厂固件、分区和配置备份，并确认有串口或救砖方式。

## 八、如何确认软件包名称

不要仅凭 LuCI 菜单名称猜包名。可以使用以下方法：

1. 在 ImmortalWrt 源码仓库中搜索插件目录或 `Makefile`。
2. 查看软件包 `Makefile` 中的 `define Package/软件包名称`。
3. 在本地源码运行 `make menuconfig`，按 `/` 搜索软件包。
4. 查看上一次 Actions 日志中的 package/config 提示。

如果 `make defconfig` 后出现找不到符号，或编译日志提示 package 不存在，通常说明该包不在 24.10 feeds 中、名称写错，或它只支持其他分支。

## 九、常见问题

### 修改 package.conf 后没有出现插件

- 检查包名是否准确；
- 检查插件是否存在于 ImmortalWrt 24.10 或 `01_prepare.sh` 添加的第三方源码中；
- 检查是否只有后端程序而没有 `luci-app-*` 网页界面；
- 查看 `Prepare` 和 `Compile` 步骤日志。

### 下载阶段失败

工作流会以较低并发自动重试下载，并缓存 `openwrt/dl`。如果只有一个变种失败，使用手动单变种方式重跑即可，不必重跑全部固件。

### 编译提示磁盘空间不足

大型 Rust、Go 插件和多套代理核心最容易增加磁盘压力。优先减少不使用的代理核心或可选组件，不要盲目把所有 Passwall 组件全部打开。

### Actions 成功但没有 Release

检查 **Settings > Actions > General > Workflow permissions** 是否设置为 **Read and write permissions**，再查看 `Create release` 步骤日志。

### Wi-Fi 菜单或无线功能异常

本项目使用上游 24.10 MT798x `defconfig` 自带的 MTK Wi-Fi 方案。增减普通 LuCI 插件时不要替换 Wi-Fi 驱动、`mtwifi-cfg`、`wifi-profile` 或 `/sbin/wifi` 相关文件。Wi-Fi 修改应先用单个变种测试，不能只以“编译成功”判断固件可用。

## 十、重要提醒

- GitHub Actions 显示编译成功，只能证明源码完成构建，不能代替真机测试。
- 每次大幅修改插件后，先编译一个与自己设备匹配的变种并测试。
- 不熟悉的内核模块、网络栈、Wi-Fi 驱动和分区组件不要删除。
- 固件刷写存在风险，操作前确认设备型号与镜像完全匹配。
