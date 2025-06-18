---
title: Openwrt x86扩容根目录
slug: openwrt-expand-rootfs
date: 2025-06-16T09:39:10+08:00
# lastmod: 2025-06-16T09:39:10+08:00 # Last modification date
tags:
  - OpenWrt
categories:
  - 软路由
# draft: true
---

仅限`x86-64-generic-ext4-combined-efi`版本
## 方法1
```shell
opkg update
opkg install parted losetup resize2fs
BOOT="$(sed -n -e "\|\s/boot\s.*$|{s///p;q}" /etc/mtab)"
DISK="${BOOT%%[0-9]*}"
PART="$((${BOOT##*[^0-9]}+1))"
echo fix | parted -l ---pretend-input-tty
parted -s ${DISK} resizepart ${PART} 100%
ROOT="${DISK}${PART}"
LOOP="$(losetup -f)"
losetup ${LOOP} ${ROOT}
resize2fs -f ${LOOP}
reboot
```
## 方法2
```shell
# Install packages
opkg update
opkg install parted losetup resize2fs blkid
 
# Download expand-root.sh
wget -U "" -O expand-root.sh "https://openwrt.org/_export/code/docs/guide-user/advanced/expand_root?codeblock=0"
 
# Source the script (creates /etc/uci-defaults/70-rootpt-resize and /etc/uci-defaults/80-rootpt-resize, and adds them to /etc/sysupgrade.conf so they will be re-run after a sysupgrade)
. ./expand-root.sh
 
# Resize root partition and filesystem (will resize partiton, reboot resize filesystem, and reboot again)
sh /etc/uci-defaults/70-rootpt-resize
```

推荐方法2，方法2每次更新也会自动运行，全自动。
