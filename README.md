# surge-reject-rules（私有）

Surge 拦截规则库。与直连库 [surge-direct-rules] 配套使用。整理自个人现有配置，2026-11-21。

## custom/

| 文件 | 内容 | 策略形式 |
| --- | --- | --- |
| reject-inline-ads.list | 酷狗 P2P/广告、字节系广告关键字、个推/友盟，以及一份按广告网络逐家列明的拦截表（AdColony、AppsFlyer、Criteo、Flurry、InMobi、Tapjoy 等数百条） | 源文件中每条自带 `,REJECT`，可直接贴入配置的 [Rule] 段 |
| reject-ads-domains.list | 另一份广告/追踪域名表（偏国内 SDK：穿山甲、快手、百度、腾讯、阿里、个推/友盟/OpenInstall 等），含一段 HTTPDNS 端点与搜狗广告 | 未带策略，用作 RULE-SET 时指定 REJECT |
| reject-pcdn.list | PCDN 拦截：哔哩哔哩 PCDN（PPio/派欧云、臻选云等）、B 站遥测域名、PCDN 关键字与 `PROTOCOL,STUN` | 未带策略（源文件按段落标题归类），用作 RULE-SET 时指定 REJECT |

## 整理说明

- reject-ads-domains.list 中源文件有若干行行尾多一个逗号（如 `DOMAIN,xxx,`、`IP-CIDR,x,no-resolve,`），在 Surge 中可能被当作空参数，已在入库时去掉多余逗号，其余与源文件一致。
- reject-pcdn.list 中 `DOMAIN-SUFFIX,zhenxyun.cn` 在源文件出现两次，已去重。
- reject-inline-ads.list 与 reject-ads-domains.list 是同一批广告域名的两种形态（一个内联 REJECT、一个规则集形态），内容大量重叠，实际使用选其一即可。
- 与直连库的冲突：`inmobi.com`、`kochava.com`、`mobilefuse.com`、`crashlytics.com` 相关域名在直连库 Talkatone 段也存在。Surge 先匹配先执行，效果取决于配置中 RULE-SET 的引用顺序；冲突按源文件原样保留。

## 重整化记录（2026-11-21）

全库做过一轮匹配清理（脚本 normalize.py，可重复运行）：删除精确重复、被 DOMAIN-SUFFIX 父后缀覆盖的域名（如酷狗段同时存在 DOMAIN 与 DOMAIN-SUFFIX 两份时只留后缀版）、被更短关键字包含的 DOMAIN-KEYWORD。结果（规则条数）：

| 文件 | 清理前 | 清理后 |
| --- | --- | --- |
| reject-inline-ads.list | 444 | 222 |
| reject-ads-domains.list | 535 | 403 |
| reject-pcdn.list | 80 | 79 |

处理顺序为 inline → domains → pcdn：domains 表中已被 inline 表覆盖的条目已删除。每 15 天随直连库一并复查上游与重整化。
