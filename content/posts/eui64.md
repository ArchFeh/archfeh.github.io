---
title: Win11配置EUI64 SLAAC IPv6地址并使用IPv6公网访问
summary: Win11开启Eui64，OpenWrt开启IPv6访问，使用IPv6 RDP连接
slug: eui64
date: 2025-06-25T21:23:16+08:00
# lastmod: 2025-06-25T21:23:16+08:00 # Last modification date
tags:
  - OpenWrt
categories:
  - 软路由
# draft: true
---

使用OpenWrt开启了SLAAC EUI64后缀后，发现ios/mac/windows设备的后缀都并不遵循eui64后缀规则。搜索后才发现，这些系统都默认不开启eui64后缀，因为存在一定安全隐患。

我为了在openwrt里放行固定后缀的ipv6地址，才想着开启eui64，方便直接进行公网ipv6的rdp访问。操作如下：

首先使用管理员权限打开PowerShell，输入Get-NetIPv6Protocol回车可以看到当前的IPv6设置，使用Set-NetIPv6Protocol可以修改设置。

```powershell
C:\Users\archf> Set-NetIPv6Protocol -UseTemporaryAddresses Disabled
C:\Users\archf> Set-NetIPv6Protocol -RandomizeIdentifiers Disabled
```

然后在openwrt防火墙里添加通信规则，目标地址填eui64后缀
