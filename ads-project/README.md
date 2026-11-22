# 广告拦截项目（ads-project）

拦截库的第二个项目：汇总 7 个广告拦截上游 + 用户粘贴规则 + 用户旧合并表「碎」的独有部分，合并成一份 DOMAIN-SET 成品与一份附加 RULE-SET。每 15 天与基础库一并更新（重新下载上游、比较新增、运行 build-ads-merged.py 重新合并）。

## 在 Surge 中怎么引用（推荐方式）

```
[Rule]
DOMAIN-SET,<ads-domain-set-part1.txt 的地址>,REJECT
DOMAIN-SET,<ads-domain-set-part2.txt 的地址>,REJECT
DOMAIN-SET,<ads-domain-set-part3.txt 的地址>,REJECT
RULE-SET,<ads-extra-ruleset.list 的地址>,REJECT
```

- `merged/ads-domain-set-part1~3.txt`（合计 254,147 条，2026-11-22 更新）：纯域名部分。带点前缀 `.example.com` = 该域名及全部子域名（等价 DOMAIN-SUFFIX）；不带点 = 精确匹配（等价 DOMAIN）。DOMAIN-SET 专为十万级大表设计，Surge 会预建索引，查询速度不随条数线性下降。因 GitHub 连接器对单次推送体积有限制，域名集按每份 ≤110,000 条切分为 3 份，引用时三行都要写。
- `merged/ads-extra-ruleset.list`（681 条规则）：DOMAIN-SET 表达不了的类型——DOMAIN-KEYWORD 119 条、DOMAIN-WILDCARD 410 条、IP-CIDR(6) 109 条、URL-REGEX/USER-AGENT/逻辑规则等 43 条。
- 注意：同一个文件不能同时被 RULE-SET 和 DOMAIN-SET 引用；本项目的两个成品文件是分开的，不冲突。

## upstream/（上游快照，2026-11-21 下载）

| 文件 | 来源与格式 | 内容 |
| --- | --- | --- |
| adrules-surge-part1~3.conf | adrules.top，Surge 规则集格式（原文件因体积切分） | 聚合型大表：206,201 条 DOMAIN-SUFFIX + 409 WILDCARD + 24 KEYWORD |
| loyalsoldier-reject-part1~3.txt | Loyalsoldier，Surge 规则集格式（原文件因体积切分） | 201,108 条 DOMAIN-SUFFIX（project 1 里 direct 表的拦截侧对应物） |
| anti-ad-surge.txt | anti-AD，Surge 规则集格式 | 102,741 条 DOMAIN-SUFFIX |
| anti-ad-surge2.txt | anti-AD，DOMAIN-SET 格式 | 与 anti-ad-surge.txt 条目完全相同（已逐条核对），只是换成点前缀写法 |
| fmz200-rejectAd.list | fmz200/奶思，Loon 规则（与 Surge 语法兼容） | 针对 App/小程序广告：2,196 后缀 + 329 精确 + 98 关键字 + 30 IP 等 |
| fmz200-qx-filter.list | fmz200/奶思，QuantumultX 格式 | 同上内容的 QX 版（host/host-suffix/host-keyword），合并时已转换类型名 |
| dler-adblock.list | dler-io/Rules，Surge 规则集格式 | 7,177 后缀 + 57 IP + 41 URL-REGEX + 35 精确等 |

## 合并与重叠（2026-11-21 实测）

- 七源 + 粘贴规则共有不重复后缀域名 246,911 个；删去被父后缀覆盖的 5,612 个后，域名集成品 241,413 条。
- adrules ∩ Loyalsoldier reject = 176,169 条；adrules ∩ anti-AD = 82,939 条（约 anti-AD 的 81%）；anti-AD ∩ Loyalsoldier = 78,339 条。三家互有独占部分，谁也不是谁的子集，所以合并保留全部来源。
- 用户粘贴规则的 437 条域名条目 100% 已被上游合并集覆盖（其 23 条 DOMAIN-KEYWORD 已并入附加规则集）；custom/pasted-ads.list 仅作原始对照保留。
- 2026-11-22 并入第八个来源：用户旧合并拦截表「碎」（617,531 行）的独有部分——其 234,438 个唯一后缀中，12,759 个未被原合并集覆盖，另有精确域名 1 条、关键字 8 条、通配 1 条，提取为 custom/legacy-sui.list 作为固定来源参与构建。合并后：不重复后缀 259,670 个，删去被父后缀覆盖的 5,638 个，得后缀条目 254,032 条 + 精确条目 115 条，成品合计 254,147 条。
