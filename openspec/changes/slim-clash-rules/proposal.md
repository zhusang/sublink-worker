## Why

当前 `UNIFIED_RULES` 定义了 17 条规则，`balanced` 预设包含 8 条，`comprehensive` 预设使用全部 17 条。许多规则的分类粒度过细（如 Gaming、Education、Financial、Cloud Services 单独成组），实际使用中大部分用户只需要"国内直连 + 常用服务分流 + 其余代理"的简洁方案。过多的规则组导致：
1. 生成的配置文件臃肿，rule-provider 数量多、启动时下载慢
2. 策略组过多，客户端 UI 中选择困难
3. 维护成本高，每加一条规则都要同步 Sing-Box / Clash / Surge 三端

## What Changes

- **合并细分规则**：将 Gaming、Education、Financial、Cloud Services 合并到 `Non-China` 大类，不再单独成组
- **精简 UNIFIED_RULES**：从 17 条精简到 10 条核心规则
- **重新设计预设方案**：
  - `minimal`：3 条（CN 直连 + Private + Non-China）— 保持不变
  - `balanced`：6 条（精选高频规则）— 从 8 条减到 6 条
  - `comprehensive`：10 条（全部保留的规则）— 从 17 条减到 10 条
- **保留的 10 条规则**：Ad Block, AI Services, Bilibili, Youtube, Google, Private, Location:CN, Telegram, Github, Non-China
- **移除的 7 条规则**：Microsoft, Apple, Social Media, Streaming, Gaming, Education, Financial, Cloud Services
  - Microsoft 和 Apple 服务在国内有 CDN，大部分场景不需要单独分流
  - Social Media / Streaming 的细分场景可通过 customRules 自行添加
  - Gaming / Education / Financial / Cloud Services 属于低频使用的细分

## Capabilities

### New Capabilities
- `slim-rules`: 精简统一规则集定义和预设方案，减少默认规则数量

### Modified Capabilities

## Impact

- **`src/config/rules.js`**：修改 `UNIFIED_RULES` 数组和 `PREDEFINED_RULE_SETS` 预设
- **前端 UI**：`src/components/Form.jsx` 中的规则选择列表会自动减少（读取 UNIFIED_RULES 生成）
- **API 兼容性**：使用 `selectedRules` JSON 数组指定已移除规则名称的请求将静默忽略（现有行为，`getOutbounds` 只返回匹配 UNIFIED_RULES 的规则）
- **无 Breaking Change**：`customRules` 参数不受影响，用户仍可通过自定义规则添加任意分流
