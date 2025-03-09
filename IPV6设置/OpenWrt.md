# OpenWrt IPv6 设置方案

## 前言
   本方案设置以 OpenWrt 主路由拨号环境设置，使用固件为[ImmortalWrt](https://github.com/immortalwrt/immortalwrt)。

   本方案来源[Aethersailor](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenWrt-IPv6-%E8%AE%BE%E7%BD%AE%E6%96%B9%E6%A1%88)。

## IPv6 设置方案

### OpenWrt

**1. Dnsmasq 设置**

* 关闭 Dnsmasq 的“过滤 IPv6 AAAA 记录”功能。

  如果不关闭此项，Dnsmasq 解析的地址中不会返回 IPv6 地址，也就无法访问 IPv6 网站。
