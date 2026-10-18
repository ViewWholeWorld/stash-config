# Stash 配置

公开主配置与本地私密覆写配合使用。仓库仅保存无真实密钥的配置和占位示例。

## 文件

| 文件 | 用途 |
| --- | --- |
| `Stash.yaml` | 通用主配置、业务策略组、DNS、公共规则集和 GFWList 补充分流 |
| `local.example.stoverride` | 本地私密覆写的填写示例，不含真实凭据 |

真实的 `本地私密设置.stoverride` 单独保存，不在此仓库中。

## 导入

1. 在 Stash 中先导入并启用单独提供的 `本地私密设置.stoverride`。它提供你自己的订阅集、`静态IP` 与 Tailscale 节点，以及个人内网和自定义直连规则。订阅集可以任意命名，也可以同时配置多个。
2. 从以下地址导入新的主配置（仓库须公开，才能匿名下载）：

   `https://raw.githubusercontent.com/ViewWholeWorld/stash-config/main/Stash.yaml`

3. 切换到新主配置时，停用原来的静态 IP、Tailscale、DNS、苹果代理、路由规则、远程代理六份覆写，仅保留新的本地私密覆写，避免数组重复合并和旧策略组覆盖。
4. 连接后检查订阅是否加载，核对 AI、社交、Emby 等策略组的选择。主配置保留原静态 IP 覆写里的候选顺序；设备原有选择不保证在切换配置后完全继承。

主配置依赖本地覆写提供节点与订阅集，不能单独作为完整配置启动。示例中的占位地址和密钥不可直接使用。新设备需要单独导入真实本地覆写；日常更新主配置无需重新填写凭据。

## 整理内容

- 合并原主配置、DNS 加密、苹果代理和链式代理策略组；个人内网规则及自定义直连规则留在本地私密覆写。
- 主配置不使用 `use` 绑定特定订阅集；Manual、Emby 和各地区组通过 `include-all: true` 自动纳入所有代理和订阅集，不依赖订阅集的名称或数量。
- 地区组保留地区名称和地区代码筛选及自动测速；本地 `静态IP` 与 `beans` 不匹配这些地区筛选，因此不会进入链式入口。Manual 和 Emby 的自动节点列表也会包含本地节点。
- 苹果服务按原苹果代理覆写交给 `Manual`，合并重复的 Apple 规则集声明。
- 静态 SOCKS5 凭据、订阅地址和个人内网规则保留在本地文件；Tailscale 节点省略 `auth-key` 和 `hostname`，在每台设备内独立登录。
- 原 `fake-ip-filter` 保留，包括其中的 `*`，此次不改变 DNS 过滤语义。
- 公共业务规则沿用原 Repcz 链接，每天更新；GFWList 使用 Loyalsoldier 的域名转换版，每 6 小时检查更新。

规则先匹配本地覆写中的个人内网和自定义直连规则，再匹配主配置中的苹果服务和原有业务分流；GFWList 位于 LAN 规则之后、`GEOIP,CN,DIRECT` 和 `MATCH,Final` 之前。GFWList 命中后交给独立的 `GFWList` 策略组，可单独选择 Global、Manual、静态 IP、地区组或 DIRECT；默认首选 Global，保持原有分流行为。

## 任意订阅集名称与多个机场

主配置中的动态节点组使用 `include-all: true`，不会引用任何固定的订阅集名称。你可以在本地覆写中使用 `ProviderA`、`ProviderB` 或自己喜欢的其他名称；这些名称不需要与主配置同步。

```yaml
proxy-providers: #!replace
  ProviderA:
    url: https://example.invalid/your-first-subscription
    interval: 86400
  ProviderB:
    url: https://example.invalid/your-second-subscription
    interval: 86400
```

地区组根据实际节点名称筛选；节点名称没有地区标识时，仍可在 Manual 或 Emby 的完整列表中手动选择。本地节点 `静态IP`、`beans` 和策略组名称是配置内部引用，保留现有名称；不要把静态链式代理改成带地区标识的名字后直接纳入地区组，否则可能形成循环。

## 多台设备使用 Tailscale

本地覆写中的 Tailscale 节点只保留：

```yaml
proxies:
  - name: beans
    type: tailscale
```

每台设备分别导入本地覆写与主配置后，在 Stash 的代理列表打开 `beans` 节点菜单，进入 `Tailscale` 页面，点击开始认证并完成登录。认证身份保存在各设备本地，已认证的节点通常无需在每次更新配置后重新登录。交互登录需要 iOS/tvOS 3.4+ 或 macOS 4.2+。

`name: beans` 是本配置中的节点标签，可以在不同设备共用。省略 `hostname` 后，Stash 使用系统报告的设备主机名；系统没有提供时，由控制服务器命名。配置不再把所有设备强制命名为同一个 hostname。如果几台设备本身的系统名称相同，可给系统设置不同名称，或仅在各设备本地覆写的 Tailscale 节点中指定不同 `hostname`，例如 `stash-iphone`、`stash-ipad`、`stash-mac`。

不要在共享配置中重新添加 `auth-key` 或固定的 `hostname`。支持参数及登录流程见 [Stash Tailscale 文档](https://stash.wiki/proxy-protocols/proxy-types#tailscale)。

## 维护

- 修改策略组或规则顺序：编辑 `Stash.yaml`，然后在 Stash 更新该配置。
- 修改真实订阅或静态 SOCKS5 凭据：只编辑设备中的本地私密覆写。
- 重命名订阅集或新增机场：只在本地覆写的 `proxy-providers` 下修改或新增条目，主配置无需修改；不上传真实订阅链接。
- 不将原始附件或导出的完整配置提交到仓库；`.gitignore` 仅防止常见误提交，不会清除已经进入历史的秘密。

## 来源

基础配置和业务规则源自 [Repcz/Tool](https://github.com/Repcz/Tool)，保留原作者来源；图标沿用上传配置中的原链接。
GFWList 域名转换版源自 [Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules)。它是域名列表，不完整保留 AutoProxy 的 URL 路径、关键词和正则语义。

本次已验证 YAML、覆写合并后的引用、链式代理依赖及真实敏感值隔离；未在实际 Stash 客户端连接测试。

