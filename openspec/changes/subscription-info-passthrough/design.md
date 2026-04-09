## Context

`subscription-userinfo` 是订阅服务的标准响应头，格式为：
```
subscription-userinfo: upload=1234; download=5678; total=10737418240; expire=1704067200
```
客户端（Clash/Surge/Singbox）依赖此头显示流量用量和到期信息。

当前 `httpSubscriptionFetcher.js` 的两个 fetch 函数只返回解析后的内容，不返回响应头。数据流：
```
上游订阅 → fetchSubscription/fetchSubscriptionWithFormat → parseCustomItems → Builder → 端点响应
```

## Goals / Non-Goals

**Goals:**
- 在 fetch 层捕获 `subscription-userinfo` 头
- 在 Builder 层聚合多个上游的 userinfo
- 在所有端点的响应头中返回真实数据
- 支持用户通过前端 UI 手动设置自定义 subscription-userinfo

**Non-Goals:**
- 不在前端 UI 上展示流量信息（仅提供输入）
- 不支持非标准的 userinfo 格式

## Decisions

### 1. 聚合策略

**决策**：多个上游订阅时，`upload`/`download`/`total` 求和，`expire` 取最早（最小值）。

**理由**：用户关心的是"总共用了多少、还剩多少、最早什么时候到期"。求和反映总使用量，最早到期反映最紧迫的续费需求。

### 2. 数据流设计

**决策**：在 `BaseConfigBuilder` 上新增 `subscriptionUserinfo` 属性，`parseCustomItems()` 中收集并聚合。端点从 `builder.subscriptionUserinfo` 读取。

**理由**：复用现有的 Builder 管道，不需要修改端点的参数签名。`/xray` 端点不使用 Builder，需要单独处理。

### 3. 返回格式

**决策**：原样透传标准格式 `upload=N; download=N; total=N; expire=N`，缺失字段不填充。

**理由**：保持与上游一致，客户端自行处理缺失字段。

### 4. 手动自定义优先级

**决策**：端点接受 `sub_info` 查询参数，优先级为 `sub_info` 参数 > 上游透传聚合值 > 不返回。

**理由**：用户可能自建节点没有上游订阅头，需要手动设置让客户端显示信息。手动值覆盖自动值符合直觉。

**前端**：在通用设置区域加一个输入框，placeholder 示例格式，值保存到 localStorage。submitForm 时作为 `sub_info` 参数传递。

## Risks / Trade-offs

- **[多订阅求和不精确]** → 不同上游可能有不同计费周期，求和只是近似 → 但对大多数用户场景够用
- **[/xray 端点]** → 不使用 Builder，需要在 fetch 循环中单独收集 userinfo → 作为独立任务处理
