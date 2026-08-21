# luci-app-netcontrol

OpenWrt 上网管控插件，基于 nftables 限制局域网设备的上网时间与时长，通过 LuCI Web 界面管理。

## 功能

三种控制方式，按设备逐条配置：

| 模式 | 说明 |
|---|---|
| 时间段控制 | 只在指定时段内允许上网 |
| 时长控制 | 每个重置周期（每日/每周/每月/不重置）内累计允许上网 N 分钟 |
| 组合控制 | 时段与时长同时生效 |

其他特性：

- 目标地址支持单个 IP、CIDR 网段、IP 范围、MAC 地址、IPv6。按 MAC 配置时设备换 IP 仍能跟住。
- 时长按设备**开机在线**累计：关机时段不计入配额（判据是设备有没有在发包，关机的设备一个包都发不出来）。
- 已用时长定期落盘至 `/etc/netcontrol/`，重启路由器不会清零。
- 可选强控制模式：同时拦截发往路由器自身的流量（被拦设备连 LuCI 界面也进不去）。
- 界面提供实时状态表，可单独重置某台设备的已用时长。
- 可识别并拦截使用随机 MAC 的设备：手机、电脑每次联网换一个 MAC 会让按 MAC 配的规则失效，
  开启后这类设备一律禁止上网，另设放行名单。

## 依赖

| 包 | 说明 |
|---|---|
| `nftables` | 防火墙规则引擎（必需） |
| `conntrack` | 拦截生效时断开设备的现有连接（必需） |

## 安装

在 OpenWrt 源码根目录执行：

```bash
echo 'src-git netcontrol https://github.com/rule2c/luci-app-netcontrol.git' >> feeds.conf.default
./scripts/feeds update netcontrol
./scripts/feeds install -a -p netcontrol
```

然后在 `make menuconfig` 中勾选 `LuCI → Applications → luci-app-netcontrol`，编译固件。

## 从 luci-app-timecontrol 迁移

本插件由 `luci-app-timecontrol` 重构而来。安装时会自动迁移旧配置：设备规则（含地址字段
`mac` → `target` 的改名）、已累计的上网时长一并搬迁，随后停用旧服务，避免两套规则互相
拆对方的 nftables 表。迁移只执行一次；若新配置里已有设备规则，则视为用户已自行配置，不作覆盖。

## 命令行

```
netcontrol diagnose       # 系统诊断
netcontrol status         # 查看当前 nftables 规则
netcontrol test           # 地址解析自测
netcontrol-log realtime   # 实时日志（彩色高亮）
```

## 构建

```bash
make package/feeds/netcontrol/luci-app-netcontrol/compile V=s
```

shell 脚本的目标环境是 BusyBox ash，改动后用 `sh -n` 检查语法，不要使用 bash 特有语法。

## 许可证

GPL-2.0，见 [LICENSE](LICENSE)。
