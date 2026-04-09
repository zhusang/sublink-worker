## ADDED Requirements

### Requirement: Slim unified rules set
系统 SHALL 将 `UNIFIED_RULES` 精简为以下 10 条核心规则：
1. Ad Block
2. AI Services
3. Bilibili
4. Youtube
5. Google
6. Private
7. Location:CN
8. Telegram
9. Github
10. Non-China

系统 SHALL 移除以下 7 条规则：Microsoft, Apple, Social Media, Streaming, Gaming, Education, Financial, Cloud Services。

#### Scenario: UNIFIED_RULES 数量正确
- **WHEN** 读取 `UNIFIED_RULES` 数组
- **THEN** 数组长度 SHALL 为 10

#### Scenario: 移除的规则不再存在
- **WHEN** 在 `UNIFIED_RULES` 中查找 `Microsoft`、`Apple`、`Social Media`、`Streaming`、`Gaming`、`Education`、`Financial`、`Cloud Services`
- **THEN** 均 SHALL 返回未找到

#### Scenario: 保留规则的 site_rules 和 ip_rules 不变
- **WHEN** 读取保留的 10 条规则
- **THEN** 每条规则的 `site_rules` 和 `ip_rules` 数组 SHALL 与精简前完全一致

### Requirement: Updated predefined rule sets
系统 SHALL 更新预设方案如下：
- `minimal`：`['Location:CN', 'Private', 'Non-China']`（不变）
- `balanced`：`['Location:CN', 'Private', 'Non-China', 'Google', 'Telegram', 'AI Services']`
- `comprehensive`：包含全部 10 条规则的名称

#### Scenario: minimal 预设不变
- **WHEN** 读取 `PREDEFINED_RULE_SETS.minimal`
- **THEN** SHALL 等于 `['Location:CN', 'Private', 'Non-China']`

#### Scenario: balanced 预设精简为 6 条
- **WHEN** 读取 `PREDEFINED_RULE_SETS.balanced`
- **THEN** SHALL 等于 `['Location:CN', 'Private', 'Non-China', 'Google', 'Telegram', 'AI Services']`

#### Scenario: comprehensive 预设为全部 10 条
- **WHEN** 读取 `PREDEFINED_RULE_SETS.comprehensive`
- **THEN** SHALL 包含全部 10 条规则名称，且长度为 10

### Requirement: Backward compatible selectedRules handling
使用已移除规则名称的 `selectedRules` 请求 SHALL 不会导致错误。

#### Scenario: 请求包含已移除规则名称
- **WHEN** API 接收到 `selectedRules=["Google","Microsoft","Gaming"]`
- **THEN** 系统 SHALL 仅处理 `Google`（仍存在于 UNIFIED_RULES），静默忽略 `Microsoft` 和 `Gaming`
- **AND** 响应状态码 SHALL 为 200

#### Scenario: 旧的 balanced 预设名称仍可用
- **WHEN** API 接收到 `selectedRules=balanced`
- **THEN** 系统 SHALL 使用新的 balanced 预设（6 条规则）生成配置
- **AND** 响应状态码 SHALL 为 200

### Requirement: Generated rule sets reflect slim rules
规则集生成器（Sing-Box rule_set、Clash rule-provider、Surge RULE-SET）SHALL 仅包含保留的 10 条规则对应的远程规则集引用。

#### Scenario: Clash rule-provider 数量减少
- **WHEN** 使用 `comprehensive` 预设调用 `generateClashRuleSets`
- **THEN** 生成的 site_rule_providers 和 ip_rule_providers 的总键数 SHALL 少于精简前
- **AND** 不 SHALL 包含 `microsoft`、`apple`、`facebook` 等已移除规则的 provider

#### Scenario: Sing-Box rule_set 数量减少
- **WHEN** 使用 `comprehensive` 预设调用 `generateRuleSets`
- **THEN** 生成的 site_rule_sets 和 ip_rule_sets 的总数 SHALL 少于精简前
