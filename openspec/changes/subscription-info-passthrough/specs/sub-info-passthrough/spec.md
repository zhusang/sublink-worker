## ADDED Requirements

### Requirement: Capture upstream subscription-userinfo header
`fetchSubscription()` 和 `fetchSubscriptionWithFormat()` SHALL 在获取上游订阅时提取 `subscription-userinfo` 响应头，并在返回值中包含该信息。

#### Scenario: 上游返回 subscription-userinfo 头
- **WHEN** 上游订阅响应包含 `subscription-userinfo: upload=100; download=200; total=1000; expire=1704067200`
- **THEN** 返回值 SHALL 包含 `subscriptionUserinfo` 字段，值为原始头字符串

#### Scenario: 上游未返回 subscription-userinfo 头
- **WHEN** 上游订阅响应不包含 `subscription-userinfo` 头
- **THEN** 返回值的 `subscriptionUserinfo` 字段 SHALL 为 `null`

### Requirement: Aggregate multiple upstream userinfo
`BaseConfigBuilder` SHALL 将多个上游订阅的 userinfo 聚合为单一值，存储在 `this.subscriptionUserinfo` 属性上。

#### Scenario: 单个上游订阅
- **WHEN** 只有一个上游订阅返回了 `subscription-userinfo: upload=100; download=200; total=1000; expire=1704067200`
- **THEN** `builder.subscriptionUserinfo` SHALL 等于 `upload=100; download=200; total=1000; expire=1704067200`

#### Scenario: 多个上游订阅
- **WHEN** 上游 A 返回 `upload=100; download=200; total=1000; expire=1704067200`
- **AND** 上游 B 返回 `upload=50; download=300; total=2000; expire=1701388800`
- **THEN** `builder.subscriptionUserinfo` SHALL 等于 `upload=150; download=500; total=3000; expire=1701388800`（upload/download/total 求和，expire 取最早）

#### Scenario: 无上游返回 userinfo
- **WHEN** 没有任何上游订阅返回 `subscription-userinfo` 头
- **THEN** `builder.subscriptionUserinfo` SHALL 为 `null`

### Requirement: All endpoints return subscription-userinfo header
`/singbox`、`/clash`、`/surge` 端点 SHALL 在响应中设置 `subscription-userinfo` 头（如果有值）。

#### Scenario: Builder 有 subscriptionUserinfo
- **WHEN** builder 构建完成且 `builder.subscriptionUserinfo` 不为 null
- **THEN** 响应 SHALL 包含 `subscription-userinfo` 头，值为聚合后的 userinfo 字符串

#### Scenario: Builder 无 subscriptionUserinfo
- **WHEN** builder 构建完成且 `builder.subscriptionUserinfo` 为 null
- **THEN** 响应 SHALL 不包含 `subscription-userinfo` 头

### Requirement: Remove hardcoded Surge userinfo
`/surge` 端点 SHALL 不再硬编码假的 `subscription-userinfo` 数据。

#### Scenario: Surge 端点使用真实数据
- **WHEN** 请求 `/surge` 端点
- **THEN** `subscription-userinfo` 头 SHALL 来自上游订阅的真实数据或用户手动设置，而非硬编码值

### Requirement: Custom subscription-userinfo via sub_info parameter
所有端点 SHALL 接受 `sub_info` 查询参数，允许用户手动设置 `subscription-userinfo` 头内容。优先级：`sub_info` 参数 > 上游透传 > 不返回。

#### Scenario: 用户通过 sub_info 手动设置
- **WHEN** 请求包含 `sub_info=upload%3D0%3B+download%3D1024%3B+total%3D10737418240%3B+expire%3D1735689600`
- **THEN** 响应的 `subscription-userinfo` 头 SHALL 为用户设置的值，忽略上游透传值

#### Scenario: sub_info 为空，有上游透传
- **WHEN** 请求不包含 `sub_info` 参数
- **AND** 上游订阅返回了 `subscription-userinfo` 头
- **THEN** 响应 SHALL 使用上游透传的聚合值

#### Scenario: sub_info 为空，无上游透传
- **WHEN** 请求不包含 `sub_info` 参数
- **AND** 上游订阅未返回 `subscription-userinfo` 头
- **THEN** 响应 SHALL 不包含 `subscription-userinfo` 头
