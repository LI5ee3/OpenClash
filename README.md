# OpenClash配置

## 自定义规则
* DOMAIN-SUFFIX,google.com,Proxy 匹配域名后缀(交由Proxy代理服务器组)
* DOMAIN-KEYWORD,google,Proxy 匹配域名关键字(交由Proxy代理服务器组)
* DOMAIN,google.com,Proxy 匹配域名(交由Proxy代理服务器组)
* DOMAIN-SUFFIX,ad.com,REJECT 匹配域名后缀(拒绝)
* IP-CIDR,127.0.0.0/8,DIRECT 匹配数据目标IP(直连)
* SRC-IP-CIDR,192.168.1.201/32,DIRECT 匹配数据发起IP(直连)
* DST-PORT,80,DIRECT 匹配数据目标端口(直连)
* SRC-PORT,7777,DIRECT 匹配数据源端口(直连)

## [ruleset] 部分
启用自定义规则集的总开关，设置为 true 时打开，默认为 true

overwrite_original_rules

[ ] 前缀后的文字将被当作规则，而不是链接或路径，主要包含 []GEOIP 和 []MATCH(等同于 []FINAL)。

例如：

* ruleset=🍎 苹果服务,https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Apple.list

  表示引用 https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/Apple.list 规则

  且将此规则指向 [proxy_group] 所设置 🍎 苹果服务 策略组

* ruleset=Domestic Services,clash-domain:https://ruleset.dev/clash_domestic_services_domains,86400

  表示引用clash-domain类型的 https://ruleset.dev/clash_domestic_services_domains 规则

  规则更新间隔为86400秒

  且将此规则指向 [proxy_group] 所设置 Domestic Services 策略组

* ruleset=🎯 全球直连,rules/NobyDa/Surge/Download.list

  表示引用本地 rules/NobyDa/Surge/Download.list 规则

  且将此规则指向 [proxy_group] 所设置 🎯 全球直连 策略组

* ruleset=🎯 全球直连,[]GEOIP,CN

  表示引用 GEOIP 中关于中国的所有 IP

  且将此规则指向 [proxy_group] 所设置 🎯 全球直连 策略组

* ruleset=!!import:snippets/rulesets.txt

  表示引用本地的snippets/rulesets.txt规则
  
* ruleset=💻 GitHub,clash-classic:https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/GitHub/GitHub.yaml

  ruleset=🎯 国内流量,https://raw.githubusercontent.com/cutethotw/ClashRule/main/Rule/Inside.list

  会按照顺序匹配规则, .yaml文件 需要在前面添加  clash-classic:<ins>(.list 不需要)</ins>

## [proxy_group] 部分
[ ] 前缀后的文字将被当作引用策略组

custom_proxy_group=Group_Name`url-test|fallback|load-balance`Rule_1`Rule_2`...`test_url`interval[,timeout][,tolerance]

custom_proxy_group=Group_Name`select`Rule_1`Rule_2`...

### 格式示例

* custom_proxy_group=🍎 苹果服务`url-test`(美国|US)`http://www.gstatic.com/generate_204`300,5,100

  表示创建一个叫 🍎 苹果服务 的 url-test 策略组,并向其中添加名字含'美国','US'的节点，每隔300秒测试一次，测速超时为5s，切换节点的延迟容差为100ms

* custom_proxy_group=🇯🇵 日本延迟最低`url-test`(日|JP)`http://www.gstatic.com/generate_204`300,5

  表示创建一个叫 🇯🇵 日本延迟最低 的 url-test 策略组,并向其中添加名字含'日','JP'的节点，每隔300秒测试一次，测速超时为5s

* custom_proxy_group=负载均衡`load-balance`.*`http://www.gstatic.com/generate_204`300,,100

  表示创建一个叫 负载均衡 的 load-balance 策略组,并向其中添加所有的节点，每隔300秒测试一次，切换节点的延迟容差为100ms

* custom_proxy_group=🇯🇵 JP`select`沪日`日本`[]🇯🇵 日本延迟最低

  表示创建一个叫 🇯🇵 JP 的 select 策略组,并向其中**依次**添加名字含'沪日','日本'的节点，以及引用上述所创建的 🇯🇵 日本延迟最低 策略组

* custom_proxy_group=节点选择`select`(^(?!.*(美国|日本)).*)

  表示创建一个叫 节点选择 的 select 策略组,并向其中**依次**添加名字不包含'美国'或'日本'的节点

* custom_proxy_group=🇭🇰 香港节点select(?=.(香港|HK|Hong Kong|🇭🇰|HongKong))(^(?!.(家宽|游戏|剩余|流量|2.0|2倍|2x|3.0|3倍|3x|4.0|4倍|4x)).*)

  表示创建一个叫🇭🇰 香港节点 select 策略组,并向其中依次添加名字中含有香港|HK|Hong Kong|🇭🇰|HongKong的节点但是排除家宽|游戏|剩余|流量|2.0|2倍|2x|3.0|3倍|3x|4.0|4倍|4x

* custom_proxy_group=🇸🇬 加坡节点select(?=.(新加坡|坡|狮城|SG|Singapore))^((?!(家宽|游戏|剩余|流量|2.0|2倍|2x|3.0|3倍|3x|4.0|4倍|4x)).)$

  和上一条一样的效果!! 但是这个匹配的更全面,优先使用这个,切记附带排除的前面不可以用    (新加坡|坡|狮城|SG|Singapore)    这种包含规则,必须要用正向表达式(?=.*(新加坡|坡|狮城|SG|Singapore))    才可以,不然会删除所有节点!

  (?=.*(0.5|0.5倍|0.5x))  这种最好用表达式  单纯(0.5|0.5倍|0.5x)  也可以,效果不好的时候换表达式!

  (新加坡) 改成  (^新加坡)  ^ 表示字符串的开头  所以只匹配开头.

  比如 节点名字是    SG|新加坡    (新加坡) 会匹配,  (^新加坡)不会匹配, 新加坡|SG  就会匹配了,一个是全匹配一个是开头匹配.  此规则还没验证!

**还可使用一些特殊筛选条件：**

`!!GROUPID=%n% 待转换链接中的第 n+1 条链接中包含的节点
  
`!!INSERT=%n% 配置文件中 insert_url 的第 n+1 条链接所包含的节点
  
`!!PROVIDER=%proxy-provider-name% 指定名称的proxy-provider

GROUPID 和 INSERT 匹配支持range,如 1,!2,3-4,!5-6,7+,8-

* custom_proxy_group=g1`select`!!GROUPID=0`!!INSERT=0

  表示创建一个叫 g1 的 select 策略组,并向其中依次添加订阅链接中第一条订阅链接中的所有节点和配置文件中 insert_url 中的**第一个**节点

* custom_proxy_group=g2`select`!!GROUPID=1

  表示创建一个叫 g2 的 select 策略组,并向其中依次添加订阅链接中第二条订阅链接中的所有节点

* custom_proxy_group=g3`select`!!GROUPID=!2

  表示创建一个叫 g3 的 select 策略组,并向其中依次添加订阅链接中除了第三条订阅链接之外的所有节点

* custom_proxy_group=g4`select`!!GROUPID=3-5

  表示创建一个叫 g4 的 select 策略组,并向其中依次添加订阅链接中第四条到第六条订阅链接中的所有节点

* custom_proxy_group=v2ray`select`!!GROUP=V2RayProvider

  表示创建一个叫 v2ray 的 select 策略组,并向其中依次添加订阅链接中组名（tag）为 V2RayProvider 的所有节点

  注意：此处的订阅链接指 default_url 和 &url= 中的订阅以及单链接节点（区别于配置文件中 insert_url）

  现在也可以使用2个条件组合来进行筛选，只有同时满足这2个筛选条件的节点才会被加入组内

* custom_proxy_group=g1hkselect!!GROUPID=0!!(HGC|HKBN|PCCW|HKT|hk|港)

  属于订阅链接中的第一条订阅且名字含 HGC、HKBN、PCCW、HKT、hk、港 的节点

* custom_proxy_group=g1hkselect!!GROUPID=0!!(HGC|HKBN|PCCW|HKT|hk|港)(?!.*(深港|家宽))

  属于订阅链接中的第一条订阅且名字含 HGC、HKBN、PCCW、HKT、hk、港 的节点并且排除深港\家宽

* custom_proxy_group=g1hkselect!!GROUPID=1-2!!(HGC|HKBN|PCCW|HKT|hk|港)(?!.*(深港|家宽))

  属于订阅链接中的第二\三条订阅且名字含 HGC、HKBN、PCCW、HKT、hk、港 的节点并且排除深港\家宽

* custom_proxy_group=g1hkselect!!GROUPID=!0!!(HGC|HKBN|PCCW|HKT|hk|港)(?!.*(深港|家宽))

  属于订阅链接中除了第一条订阅且名字含 HGC、HKBN、PCCW、HKT、hk、港 的节点并且排除深港\家宽

## Google Play下载问题
在 流量控制 > 绕过指定区域 IPv4 黑名单中添加如下四条域名：
```
services.googleapis.cn
googleapis.cn
xn--ngstr-lra8j.com
clientservices.googleapis.com
```
