# LicheeRV Nano 操作备份日志

日期：2026-09-28
主机：`plain@10.20.60.73`

## 主机修改

- 新建 NetworkManager 配置 `licheervnano-rndis`。
  - 固定接口：`enx02fca0a5c540`
  - 主机地址：`192.168.1.100/24`
  - 开机自动连接，IPv6 已关闭。
- 新建 `/etc/sysctl.d/99-licheervnano-rndis.conf`，永久开启 IPv4 转发。
- 新建 `/usr/local/sbin/licheervnano-rndis-nat`，配置 `192.168.1.0/24` 经 `enp5s0` 出网的 NAT 与转发规则。
- 新建并启用 `licheervnano-rndis-nat.service`，主机重启后自动恢复 NAT。
- 清理了重复的 NAT 规则；目前保留一条有效规则。
- 临时创建的无密码 sudo 配置已删除。

## SDK 修改

仓库：`/home/plain/git/LicheeRV-Nano-Build`

- LED 默认触发器改为 1 Hz 定时闪烁，并启用定时器 LED 触发器。
- RNDIS 板端地址固定为 `192.168.1.9`。
- SD 启动分区加入 `rndis.ipv4_address` 标记文件。
- 扩大 rootfs 分区，避免镜像打包溢出。
- 修改 Gadget 启动脚本，支持从启动分区读取静态 RNDIS 地址。

主要涉及：

- `build/boards/sg200x/sg2002_licheervnano_sd/sg2002_licheervnano_sd_defconfig`
- `build/boards/sg200x/sg2002_licheervnano_sd/dts_riscv/sg2002_licheervnano_sd.dts`
- `build/boards/sg200x/sg2002_licheervnano_sd/linux/sg2002_licheervnano_sd_defconfig`
- `build/boards/sg200x/sg2002_licheervnano_sd/partition/partition_sd.xml`
- `buildroot/board/cvitek/SG200X/overlay/etc/init.d/S30gadget_nic`
- `build/common_functions.sh`
- `build/tools/common/sd_tools/sd_gen_burn_image_rootless.sh`
- `build/tools/common/sd_tools/genimage_rootless.cfg`
- 新增 `build/boards/sg200x/sg2002_licheervnano_sd/rndis.ipv4_address`

## 镜像与板端

- 已构建并刷写镜像到 `/dev/sda`。
- 最终镜像：
  `/home/plain/git/LicheeRV-Nano-Build/install/soc_sg2002_licheervnano_sd/images/2026-09-28-19-54-d4003f.img`
- 板端写入持久化默认路由：`192.168.1.100`。
- 板端写入 DNS：`223.5.5.5`、`1.1.1.1`。
- 固定 RNDIS MAC，避免每次上电生成新的主机接口名。

## 验证结果

- 主机与板端互通：`192.168.1.100` <-> `192.168.1.9`。
- 板端重启后主机自动恢复 RNDIS 配置。
- 板端可访问 `223.5.5.5` 和 `baidu.com`。
- 主机 NAT 服务已启用并处于 active 状态。

说明：日志不包含密码、密钥或令牌。

## 迁移到另一台开发机（2026-10-18 补充）

当前 SDK 路径已变为 `/home/plain/git/licheeRV/LicheeRV-Nano-Build`；上文旧路径为历史记录。

### 仓库和版本

需要手动克隆的只有两个仓库，均应锁定以下提交，不要直接使用最新分支：

| 目录 | 仓库 | 当前分支 | 提交 |
| --- | --- | --- | --- |
| SDK 根目录 | https://github.com/sipeed/LicheeRV-Nano-Build.git | main | d4003f15b35d43ad4842f427050ab2bba0114fa5 |
| host-tools | https://github.com/sophgo/host-tools.git | master | 103c66f126fa98fcaa8b54f37fa06f6b293fd074 |

主 SDK 无注册子模块；`linux_5.10`、`u-boot-2021.10`、`buildroot` 等源码都由主仓库直接跟踪，不需分别克隆。`host-tools` 是独立仓库，不会随主仓库克隆自动获得，当前无已跟踪文件改动。

Buildroot 下载缓存中还存在以下有效 Git 仓库，由构建系统自动获取，不必手动放入 SDK：

| 缓存目录（相对 buildroot/dl） | 来源 | 提交 |
| --- | --- | --- |
| qt5location/git | https://invent.kde.org/qt/qt/qtlocation | ebdf5b38da6e3fe0a183702bcca395c589c459ed |
| qt5location/git/src/3rdparty/mapbox-gl-native | https://invent.kde.org/qt/qt/qtlocation-mapboxgl.git | d3101bbc22edd41c9036ea487d4a71eabd97823d |
| rtc-tools/git | https://git.kernel.org/pub/scm/linux/kernel/git/abelloni/rtc-tools.git | 33ef4aa1c92b0c92a351284d93d1ac5570de9cc7 |

此外，`buildroot/output/build` 下有 QtBase、QtCharts、QtSvg、QtImageFormats、QtLocation，以及 html5lib/testdata、NumPy/scipy-mathjax、x86-simd-sort、svml 的 `.git` 残留；这些元数据已失效，不应按独立开发仓库迁移。

Qt 依赖版本由 SDK 内的 package 配方固定：

- qt5base：`e44097b63d17ba3178a637df7fac51ddc51cb48b`
- qt5charts：`841083c263f16f8acb81e4915bf24bbed0461c9a`
- qt5svg：`a9752b00a03438de8e5b080073816e12409404b7`
- qt5imageformats：`9044695f4b1acdd7ec24e7b01f6a6ca61f982a75`
- NumPy：`1.25.0`；html5lib：`1.1`（发行源码归档，不需手动克隆）。

### 已保存的复现材料

目录：`/home/plain/ai_log/licheervnano-reproduce-2026-10-18/`

- `sdk-local-changes.patch`：相对主 SDK HEAD 的完整修改，含暂存和未暂存改动（10 个已跟踪文件）。
- `sdk-new-files.tar.gz`：4 个新增功能文件，保留权限。
- `sdk-status.txt`：导出时的工作区状态。
- `host-tools-local-changes.patch`：当前为空，说明无已跟踪文件修改。
- `host-network-config.tar.gz`：主机 sysctl、NAT 脚本和 systemd 服务备份；不含 NetworkManager 连接密钥。
- `build-environment.Dockerfile`：SDK 提供的 Ubuntu 22.04 构建依赖清单。
- `SHA256SUMS`：备份文件校验和。

已在独立临时目录验证补丁应用和新增文件解包：10 个修改文件、4 个新增文件逐一对比一致，`S31rndis_internet` 的可执行权限和 shell 语法检查通过。备份校验和全部通过；尚未在新开发机完整构建。

新增文件为 `rndis.ipv4_address`、`usb.rndis.host_mac`、`usb.rndis.dev_mac`（均位于目标 board 目录）和 overlay 的 `S31rndis_internet`。固定 MAC 支持、默认路由和 DNS 已于 9 月 29 日固化到 SDK，不再只是板端手工配置。

补丁也包含 `S77rndis` 的历史修改，但当前 Buildroot 实际使用 `S08usbdev`、`S30gadget_nic`、`S31rndis_internet`。`.orig` 备份文件不参与功能恢复，未放入新增文件归档。

### 新开发机恢复步骤

先复制上述备份目录到新机器 `~/ai_log/`，安装 Git 和构建依赖。当前机器是 Ubuntu 24.04.3 x86_64；SDK 自带 Dockerfile 基于 Ubuntu 22.04，二者不是完全相同的环境。优先按 Dockerfile 准备依赖，并额外确认 `bzip2`、`mtools`、`ripgrep` 可用；Ubuntu 24.04 不能直接照搬已移除的 `python3-distutils` 包名。

```sh
mkdir -p ~/git/licheeRV
cd ~/git/licheeRV
git clone https://github.com/sipeed/LicheeRV-Nano-Build.git
cd LicheeRV-Nano-Build
git checkout --detach d4003f15b35d43ad4842f427050ab2bba0114fa5
git clone https://github.com/sophgo/host-tools.git host-tools
git -C host-tools checkout --detach 103c66f126fa98fcaa8b54f37fa06f6b293fd074

backup="$HOME/ai_log/licheervnano-reproduce-2026-10-18"
(cd "$backup" && sha256sum -c SHA256SUMS)
git apply --check "$backup/sdk-local-changes.patch"
git apply "$backup/sdk-local-changes.patch"
tar -xzf "$backup/sdk-new-files.tar.gz"
git diff --check
bash -lc 'source build/cvisetup.sh && defconfig sg2002_licheervnano_sd && build_all'
```

以上 checkout 只用于新克隆目录；不要在已有改动的目录直接切换版本。使用完整克隆以便取到指定历史提交。首次构建依赖网络下载，不保证上游站点永远可用。

建议另外用 `rsync -a` 复制旧机器的 `buildroot/dl/` 下载缓存（当前约 1.3 GB），必要时复制完整 `host-tools/`（约 9.3 GB）；不要复制含旧绝对路径的 `buildroot/output/`，应在新机重新构建。未备份所有 apt 包版本及全部工具链文件，因此这是源码与配置复现方案，不保证镜像逐字节一致。

### 新主机 USB 联网

SDK 已固定板端 `192.168.1.9`，网关 `192.168.1.100`，DNS `223.5.5.5` / `1.1.1.1`；主机侧配置不会随 SDK 自动恢复。

```sh
sudo nmcli connection add type ethernet ifname enx02fca0a5c540 \
  con-name licheervnano-rndis ipv4.method manual \
  ipv4.addresses 192.168.1.100/24 ipv4.never-default yes \
  ipv6.method disabled connection.autoconnect yes
sudo tar -xzf "$backup/host-network-config.tar.gz" -C /
```

启用前，把 `/usr/local/sbin/licheervnano-rndis-nat` 中的旧出口 `enp5s0` 改成新机器实际外联网卡名（用 `ip route show default` 确认），并检查 `192.168.1.0/24` 是否与现有网络冲突。确认安全后执行：

```sh
sudo sysctl -p /etc/sysctl.d/99-licheervnano-rndis.conf
sudo systemctl daemon-reload
sudo systemctl enable --now licheervnano-rndis-nat.service
systemctl is-enabled licheervnano-rndis-nat.service
systemctl is-active licheervnano-rndis-nat.service
ping -c 3 192.168.1.9
```

再登录板端验证 `ip route`、`ping -c 3 223.5.5.5`、`ping -c 3 baidu.com`。固定 MAC 默认适合单板；多块板同时接入时须分配不同 MAC/IP，并调整主机连接配置。无需恢复临时无密码 sudo 授权。

最新已刷写镜像为 `2026-09-29-09-56-d4003f.img`（SDK 的 `install/soc_sg2002_licheervnano_sd/images/` 下）。上文 9 月 28 日镜像仅是历史版本。
