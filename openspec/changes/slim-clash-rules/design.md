## Context

当前 `src/config/rules.js` 中 `UNIFIED_RULES` 定义了 17 条规则，每条规则包含 `site_rules`（域名规则集）和 `ip_rules`（IP 规则集）。这些规则被三个构建器（Singbox/Clash/Surge）共用，生成对应的 rule-set / rule-provider / RULE-SET 引用。

问题：
- `comprehensive` 预设生成 17 个策略组 + 对应的远程规则集，客户端启动时需全部下载
- 其中 Gaming、Education、Financial、Cloud Services 等低频分类实际使用率极低
- Microsoft、Apple 在国内有 CDN 节点，大部分场景不需要单独分流
- Social Media、Streaming 涵盖的服务太杂（5+ 域名规则集），不如让用户按需 customRules

## Goals / Non-Goals

**Goals:**
- 将 UNIFIED_RULES 从 17 条精简到 10 条，移除低频和可合并的规则
- 重新调整 `balanced` 预设，控制在 6 条以内
- 保持 API 向后兼容：已移除规则名称的 selectedRules 请求不会报错（静默忽略）

**Non-Goals:**
- 不修改 customRules 机制（用户自定义规则不受影响）
- 不修改规则集的远程 URL 源（ruleUrls.js 不变）
- 不修改构建器的规则生成逻辑（ruleGenerators.js 的代码逻辑不变，只是输入数据减少）
- 不新增"可扩展规则注册"等新功能

## Decisions

### 1. 移除而非合并

**决策**：直接从 UNIFIED_RULES 中删除 7 条规则，而不是将它们合并到其他规则中。

**理由**：
- 合并方案会改变已有规则的 site_rules 列表，导致"Google"组突然多出不相关的域名
- 被移除的服务大多已被 `Non-China`（geolocation-!cn）覆盖，不需要显式规则
- 保持每条规则的语义清晰：规则名 = 它实际匹配的内容

**备选方案**：创建一个 "Other" 大杂烩规则合并所有低频规则 → 拒绝，因为语义不清

### 2. 保留 Non-China 作为兜底分流

**决策**：`Non-China`（geolocation-!cn）保持不变，作为"未命中任何专用规则的非中国域名"的兜底。

**理由**：被移除的 Microsoft、Apple、Social Media 等域名本身就在 geolocation-!cn 中，移除专用规则后它们自然会被 Non-China 组捕获。

### 3. balanced 预设调整

**决策**：`balanced` 从 `['Location:CN', 'Private', 'Non-China', 'Github', 'Google', 'Youtube', 'AI Services', 'Telegram']`（8条）调整为 `['Location:CN', 'Private', 'Non-China', 'Google', 'Telegram', 'AI Services']`（6条）。

**理由**：
- Youtube 已包含在 Non-China 中，大部分用户不需要单独策略组
- Github 同理，开发者用户可通过 comprehensive 或 customRules 启用
- Google + Telegram + AI Services 是最常需要独立分流控制的服务

## Risks / Trade-offs

- **[用户期望变化]** → 升级后 comprehensive 预设从 17 组变 10 组，老用户可能困惑 → 在发布说明中明确说明变更内容和 customRules 替代方案
- **[selectedRules 兼容性]** → 使用已移除规则名称的旧短链接不会报错，但对应规则不再生效 → 这些域名会 fall through 到 Non-China 组，功能上等价
- **[测试覆盖]** → 现有测试中 `selectedRules-compatibility.test.js` 可能需要更新 → 作为实施任务处理
