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
