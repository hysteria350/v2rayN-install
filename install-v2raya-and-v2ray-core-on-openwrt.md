# OpenWrt 安装 v2rayA 和 v2ray 内核

本文主要讲解如何在 OpenWrt 上安装 v2rayA 和 v2ray 内核。

## 1. 通过 v2rayA 自建软件源安装

## 2. 通过 OpenWrt 官方软件源安装

> 用户友好的 LuCI 应用 `luci-app-v2raya` 已被上游合并，使用 OpenWrt 23.05 或更高版本的用户可以直接从软件源安装。

```bash
opkg update
opkg install luci-i18n-v2raya-zh-cn v2ray-geoip v2ray-geosite
```

> 软件包 `v2raya` 已被上游合并，使用 OpenWrt 21.02 或更高版本的用户可以直接从软件源安装。

```bash
opkg update
opkg install v2raya
```

> 由于包管理器 opkg 对依赖的处理方式欠佳，目前 v2rayA 默认依赖于 Xray-core。
>
> 如果你打算使用 V2Ray，可以在安装时加入 `--nodeps` 参数，并自行安装对应的依赖。
>
> 在同时存在 V2Ray 与 Xray 的情况下，v2rayA 将优先使用后者。

## 3. 手动安装

### 3.1. 安装 V2Ray / Xray 内核

> 软件包 `xray-core`已在 OpenWrt 21.02 及之后的版本中可用。
>
> 软件包 `v2ray-core` 已在 OpenWrt 22.03 及之后的版本中可用。

首先安装软件包 `unzip` 与 `wget`，然后从 [Github Releases](https://github.com/v2fly/v2ray-core/releases) 下载 V2Ray 核心并将其保存到 `/usr/bin/`，最后赋予二进制文件可执行权限。

例如：

```bash
opkg update; opkg install unzip wget-ssl
wget https://github.com/v2fly/v2ray-core/releases/download/v4.40.1/v2ray-linux-64.zip
unzip -d v2ray-core v2ray-linux-64.zip
cp v2ray-core/v2ray v2ray-core/v2ctl /usr/bin
chmod +x /usr/bin/v2ray; chmod +x /usr/bin/v2ctl
```

Xray 核心可以用类似的方法安装。

> **擦亮眼睛**
>
> 格外注意你的 OpenWrt 设备的架构，不要下载到不适用于你设备的版本，否则核心将无法运行。

### 3.2. 安装 v2rayA

#### 3.2.1. 安装依赖包与内核模块

OpenWrt 22.03 或更新版本：

```bash
opkg update
opkg install \
    ca-bundle \
    ip-full \
    kmod-nft-tproxy
```

OpenWrt 21.02 或更早版本：

```bash
opkg update
opkg install \
    ca-bundle \
    ip-full \
    iptables-mod-conntrack-extra \
    iptables-mod-extra \
    iptables-mod-filter \
    iptables-mod-tproxy \
    kmod-ipt-nat6
```

#### 3.2.2. 下载 v2rayA 二进制文件

从 [Github Releases](https://github.com/v2rayA/v2rayA/releases) 下载最新版本对应处理器架构的二进制文件，然后移动到 `/usr/bin/` 并赋予可执行权限：

```bash
wget https://github.com/v2rayA/v2rayA/releases/download/v2.0.5/v2raya_linux_x64_2.0.5 -O v2raya
mv v2raya /usr/bin/v2raya && chmod +x /usr/bin/v2raya
```

#### 3.2.3. 下载配置文件和服务文件

```bash
wget https://raw.githubusercontent.com/openwrt/packages/master/net/v2raya/files/v2raya.config -O /etc/config/v2raya
wget https://raw.githubusercontent.com/openwrt/packages/master/net/v2raya/files/v2raya.init -O /etc/init.d/v2raya
chmod +x /etc/init.d/v2raya
```

## 4. 启用并运行 v2rayA

```bash
uci set v2raya.config.enabled='1'
uci commit v2raya
/etc/init.d/v2raya enable
/etc/init.d/v2raya start
```

## 5. 常见故障

### 5.1. PPPoE 拨号问题

~~如果你通过 PPPoE 拨号上网，那么你可能会遇到 v2rayA 的透明代理开启一段时间后没有网络连接的故障。解决方法是，使用 v2rayA 的时候不要删除或替换“网络 > 接口”默认的 WAN 连接（该连接使用 DHCP 协议），而应该新建一个接口来进行拨号。**新建的 PPPoE 拨号接口需要添加到名为 wan 的防火墙区域。**~~

该问题已经于 v2.0.2 及之后的版本中修复。

### 5.2. 防止 DNS 污染对局域网设备不生效

编辑 "接口 -> LAN -> 使用自定义的 DNS 服务器" 为 "127.2.0.17" 即可让局域网内的其他设备也享受到 "防止 DNS 污染" 的效果

### 5.3. 部分设备无法运行

~~v2rayA 所用的[数据库模块](https://github.com/boltdb/bolt)目前不支持基于 MIPS 的芯片，这部分设备（比如一些便宜的 Wi-Fi 路由器、国产龙芯电脑等）可能无法正确初始化数据库，从而导致无法使用。~~

该问题已经于 v1.5.9.1698.1 及之后的版本中修复。

内核模块不全的操作系统将无法按预期工作，建议使用官方 OpenWrt 或者第三方发行分支 ImmortalWrt。

## 6. v2rayA 使用方法

安装并成功启动v2rayA服务后，可以使用浏览器直接访问 <http://127.0.0.1:2017> v2rayA管理页面, 具体使用方法可以访问 [v2rayA 基本使用方法](https://v2raya.net/prologue/quick-start.html)

## 7. 参考文档

1. [v2rayA for OpenWrt 仓库主页](https://github.com/v2raya/v2raya-openwrt)

2. [v2rayA OSDN 主页](https://osdn.net/projects/v2raya/)

