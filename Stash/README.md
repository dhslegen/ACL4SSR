# Stash 固定住宅 IP 链路

`us-residential-chain-ios.stoverride` 将 Decodo 固定住宅代理接入已有的 ACL4SSR 订阅，兼容 Stash iOS 3.6.5。住宅连接通过订阅中已有的 `mini` 机房节点建立。

## 手机导入

1. [点击导入覆写](https://link.stash.ws/install-override/raw.githubusercontent.com/dhslegen/ACL4SSR/master/Stash/us-residential-chain-ios.stoverride)，也可在 Stash 的覆写页面手动添加下方原始地址：

   ```text
   https://raw.githubusercontent.com/dhslegen/ACL4SSR/master/Stash/us-residential-chain-ios.stoverride
   ```

2. 在手机本地编辑该覆写，将所有 `REPLACE_WITH_RESIDENTIAL_USERNAME` 替换成原始住宅代理用户名，将所有 `REPLACE_WITH_RESIDENTIAL_PASSWORD` 替换成住宅代理密码。原始用户名不包含 `user-` 前缀及 `-ip-` 参数；模板已补齐这两部分。三个节点都必须填写。建议通过 Stash 的 YAML 编辑器修改字段；若直接编辑源码，密码中含有 YAML 特殊字符时应使用引号。
3. 确认当前订阅有名称完全一致的 `mini` 节点。若名称不同，需要修改三个住宅节点的 `dialer-proxy`，以及策略组中的 `mini` 引用。
4. 在当前订阅上启用该覆写，再到 `🏠 住宅代理` 组选择所需 IP。首次导入默认选择 `48.46.148.179`，`💬 Ai平台` 只引用住宅组。

公开模板不含真实代理凭据，未经填写无法使用。iOS 3.6.5 没有覆写参数表单，因此远程更新模板后需要检查本地凭据是否仍在；建议保留一份私人已填写文件，通过 AirDrop 或“文件”App 导入，并在手动更新时保留自己的凭据。

## 分组与链路

- `🏠 住宅代理`：手动选择三个固定住宅 IP，不自动切换至机房或直连。
- `🏢 机房代理`：使用原订阅中的 `mini`。
- `🚀 节点选择`：可选择住宅组、机房组、具体节点或原有直连选项。
- `💬 Ai平台`：跟随住宅组当前选择，只有住宅出口。

链路为：手机 → Trojan 机房上游 → HTTP/TLS 住宅入口 → 目标网站。住宅节点使用用户名中的 `-ip-` 参数指定完整出口 IP，不依赖入口端口的 IP 排序。已购 IP 被供应商替换后，需要手动更新对应节点及引用。

## 兼容范围

- 依赖原订阅已有 `mini` 节点，以及本仓库 ACL4SSR Online 系列的策略组命名和域名分流规则。该文件是覆写，不是完整订阅。
- `proxy-groups` 使用 `#!replace` 完整替换，因此原订阅新增策略组后，需要核对相应规则的目标组是否仍然存在。
- iOS 不支持 `PROCESS-NAME` / `PROCESS-PATH`，Mac 上针对 AdsPower、SunBrowser 和 Claude 的进程规则没有迁移；手机继续按订阅中的域名规则分流。
- 住宅节点保留 `tls: true` 和 `block-quic: 'on'`。HTTP 代理不能承载普通 UDP，QUIC 拦截不等于拦截所有 UDP。路由到住宅节点的普通 UDP 无法通过；这里没有为其添加机房或直连兜底，也没有新增全局 UDP 拒绝规则。
- 此覆写不修改 DNS、TUN 或 Tailscale 配置，也不新增或替换原订阅的代理节点。若同时使用本目录的 `Tailscale.stoverride`，应让 Tailscale 覆写在住宅覆写之后参与合并，确保其 `🔗 Tailscale` 策略组没有被住宅覆写的 `#!replace` 移除。启用前检查最终配置是否同时包含 `Tailscale` 节点、`🔗 Tailscale` 组及其规则引用。
- 静态检查已覆盖 YAML、策略组引用、敏感凭据和进程规则；手机真机的认证、节点连接与分流结果仍需在导入后确认。

参考：[Stash 覆写配置](https://stash.wiki/configuration/override)、[代理链](https://stash.wiki/proxy-protocols/dialer-proxy)、[规则类型](https://stash.wiki/rules/rule-types)、[URL Schema](https://stash.wiki/faq/url-schema)。
