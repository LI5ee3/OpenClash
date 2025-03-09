# OpenWrt & OpenClash IPv6 设置方案

## 前言
   本方案设置以 OpenWrt 主路由拨号环境设置，使用固件为[ImmortalWrt](https://github.com/immortalwrt/immortalwrt)。

   本方案来源[Aethersailor](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenWrt-IPv6-%E8%AE%BE%E7%BD%AE%E6%96%B9%E6%A1%88)。

## IPv6 设置方案

### OpenWrt

**1. Dnsmasq 设置**

* 关闭 Dnsmasq 的“过滤 IPv6 AAAA 记录”功能。

  如果不关闭此项，Dnsmasq 解析的地址中不会返回 IPv6 地址，也就无法访问 IPv6 网站。
  
![过滤 IPv6 AAAA 记录](https://github.com/user-attachments/assets/7351fd4a-af47-473e-84d0-b06a6c8b6929)

**2. WAN 口设置 IPv6 地址**

* 自动创建

  1.不需要新建 WAN6，已有的 WAN6 接口要删除。

  2.在 WAN 口的高级设置中，开启 IPv6 的选项，并勾选使用运行商通告的 DNS。
  
  3.禁用`IPv6 分配长度`。
  
  4.启用`委托 IPv6 前缀`(启用的话 lan 是没有 IPv6 地址的)。
  
  5.IPv6相关选项不要选择或填写，填了会获取不到地址。
  <img width="1920" alt="WAN 设置" src="https://github.com/user-attachments/assets/b2297f61-cfaa-425d-a881-ecd375254f28" />
  
  6.在`WAN 接口`的`DHCP`中检查设置，确保`DHCP > IPv6 设置`已经关闭
  <img width="634" alt="WAN DHCP" src="https://github.com/user-attachments/assets/676b09ee-e421-4c54-89cd-14ada96db872" />

  7.保存并应用设置后，你的接口界面中应该会出现一个虚拟的 wan_6接口，注意此接口是无法编辑设置的。

  8.确认红框中的`IPv6-PD`地址，获取到了这个地址才能进行下一步操作。
  <img width="695" alt="WAN6 IPv6-DP" src="https://github.com/user-attachments/assets/99ec2ecc-662e-45eb-ba3c-b5a2699b5f16" />

**3. LAN 口设置下发 IPv6 地址**

* LAN 口设置
  
   **1.`委托 IPv6 前缀`允许下级设备再划分子网，按需勾选。**
   ![LAN 设置](https://github.com/user-attachments/assets/9c980c92-afe5-4f87-98f1-d40078e8d066)

   接着设置 LAN 口的 IPv6 网络地址分配服务，让局域网设备可以取得 IPv6 地址。

   需要说明的是，IPv6 地址由前缀和后缀组成，前缀由运营商下发，后缀有两种获取 IP 的方式：

   * 1、SLAAC（无状态）：后缀由局域网设备自身生成。
  
     所有类型的设备都支持该功能。

   * 2、DHCPv6（有状态）：后缀由 OpenWrt 统一管理。
  
     安卓设备以及其他的一些设备不支持该功能。

   所以此处关闭 DHCPv6，启用 SLACC，并使用`eui64`参数来启用 EUI-64 网络地址分配方式，从而形成固定 IP。

   参考如下设置：[immortalwrt/user-FAQ/如何优雅的使用IPV6？](https://github.com/immortalwrt/user-FAQ/blob/11d154ae78f8634a5e742652100984174a8d46e9/immortalwrt%20%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%8C%87%E5%8C%97.md#5-%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E4%BD%BF%E7%94%A8ipv60)

   **2.在`LAN > 高级设置 > IPv6 后缀`中填入`eui64`。**
   ![eui64](https://github.com/user-attachments/assets/29743ccf-c2dc-476a-bc13-93c750dd2b86)

   EUI-64 可以让设备的后缀地址由 MAC 地址生成，因此可以生成唯一的后缀。

   EUI-64 网络地址分配方式的技术解释可以看 ImmortalWrt 仓库的文档：[immortalwrt/user-FAQ/关于eui64的一些说明](https://github.com/immortalwrt/user-FAQ/blob/11d154ae78f8634a5e742652100984174a8d46e9/%E5%85%B3%E4%BA%8Eeui64%E7%9A%84%E4%B8%80%E4%BA%9B%E8%AF%B4%E6%98%8E.md)

   同时我们需要禁止 OpenWrt 通告 IPv6 地址的 DNS，因为设备只需要 OpenWrt 的 IPv4 DNS 地址即可实现 IPv6 解析。

   强迫下游设备使用 OpenWrt 的 IPv4 地址（比如192.168.1.1）来解析包括 IPv6 域名在内的全部域名，可以避免很多问题。
   ![DHCPv6](https://github.com/user-attachments/assets/57dcdcaa-ad4f-48e6-a527-b67c6a1c71f6)
   ![SLAAC](https://github.com/user-attachments/assets/e5b0134b-1bdf-485d-ba59-1a0241fb4b15)

   如此设置之后，局域网支持 IPv6 的设备都将获得一个固定且唯一的 IPv6 地址，并且 IPv6 DNS 为空。

**4. 测试**

* 访问 IPv6 测试网站来验证设置是否正确：

   [https://testipv6.cn/](https://testipv6.cn/)

   正常情况下，会取得全部通过的结果
   ![ipv6test](https://github.com/user-attachments/assets/c1721bab-1ac7-4e08-94a5-5d412b8b84aa)

   至此，OpenWrt 的 IPv6 功能设置完毕。

### OpenClash

**1. IPv6 设置**

  如果节点不支持 IPv6 出站，或者 OpenWrt 没有开启 IPv6 功能，则禁用`IPv6流量代理`和`允许IPv6类型DNS解析`。
  ![OpenClash IPv6](https://github.com/user-attachments/assets/3e4944e1-b31d-4535-8aa8-2ca621fd3a45)

  如果启用了此处的 IPv6 功能，并且平时要使用 Google Play，请绕过指定区域 IPv6 黑名单中添加如下四条域名：
  ```
  services.googleapis.cn
  googleapis.cn
  xn--ngstr-lra8j.com
  clientservices.googleapis.com
  ```
  ![blacklist-ipv6](https://github.com/user-attachments/assets/a85cd667-086d-4ea8-b170-3caf7bbd4f1a)

## IPv6 如何正确设置“端口转发”

   首先明确一点，你的下游设备取得的都是公网 IPv6 地址，因此此处实际上并不需要“端口转发”功能，只需要设置对应的防火墙规则，即可实现和 IPv4 的端口转发一样的使用效果。

   虽然局域网内设备取得了 IPv6 公网地址，但是你会发现从公网虽然可以 ping 通这些地址，但是并不能直接访问这些地址的端口服务。
   
   原因在于 OpenWrt 的防火墙规则默认放行转发给下游设备的 IPv6 的 ICMP 数据包，但是并没有对其他的数据进行放行。这是一种安全的设定，可以避免下游设备在取得 IPv6 公网地址后不安全的暴露于公网环境中。

   如果你需要从公网访问 OpenWrt 的下游设备的 IPv6 地址的特定端口（比如群晖的5000端口），则需要建立相应的防火墙通信规则，对特定地址和端口进行放行。

   具体设置参考：[immortalwrt/user-FAQ/IPV6如何正确配置端口转发？](https://github.com/immortalwrt/user-FAQ/blob/main/immortalwrt%20%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E6%8C%87%E5%8C%97.md#6-ipv6%E5%A6%82%E4%BD%95%E6%AD%A3%E7%A1%AE%E9%85%8D%E7%BD%AE%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91)

   注意填写地址部分，只需要填写设备的 IPv6 地址的后16位，也就是 MAC 生成的部分，这样防火墙规则会按照地址后缀去匹配设备，无需担心运营商下发的地址前缀变动。而地址后缀是根据 MAC 生成的固定后缀。
