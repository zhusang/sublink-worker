## Why

当前从上游订阅获取代理节点时，上游返回的 `subscription-userinfo` 响应头（包含流量使用量、总量、到期时间）被完全丢弃。Surge 端点硬编码了一组假数据（`total=10737418240; expire=2546249531`），其他端点（Singbox、Clash、Xray）完全没有返回此头。

这导致客户端无法显示真实的剩余流量和到期时间，影响用户体验。

## What Changes

- **捕获上游订阅信息**：`httpSubscriptionFetcher.js` 的 `fetchSubscription()` 和 `fetchSubscriptionWithFormat()` 在获取上游订阅时，提取 `subscription-userinfo` 响应头并返回
- **透传到 Builder**：`BaseConfigBuilder.parseCustomItems()` 收集所有上游订阅的 userinfo，聚合后存储在 builder 实例上
- **所有端点返回真实头**：`/singbox`、`/clash`、`/surge`、`/xray` 端点从 builder 读取聚合后的 userinfo，设置到响应头中
- **移除 Surge 端点的硬编码假数据**
- **支持手动自定义**：前端新增 subscription-userinfo 输入框，用户可手动填写 `upload=xx; download=xx; total=xx; expire=xx`；通过 `sub_info` 查询参数传递到后端，优先级：手动设置 > 上游透传 > 不返回

## Capabilities

### New Capabilities
- `sub-info-passthrough`: 从上游订阅获取 subscription-userinfo 并透传到客户端响应头

### Modified Capabilities

## Impact

- **`src/parsers/subscription/httpSubscriptionFetcher.js`**：两个 fetch 函数的返回值增加 `subscriptionUserinfo` 字段
- **`src/builders/BaseConfigBuilder.js`**：`parseCustomItems()` 收集 userinfo，新增 `this.subscriptionUserinfo` 属性
- **`src/app/createApp.jsx`**：四个端点读取 `sub_info` 参数和 builder 的 subscriptionUserinfo，按优先级设置响应头
- **`src/components/Form.jsx`**：通用设置区域新增 subscription-userinfo 输入框
- **`src/components/formLogic.js`**：新增 `customSubInfo` 状态 + localStorage 持久化 + submitForm 传参
- **`src/i18n/index.js`**：新增相关翻译
- **多订阅聚合**：当有多个上游订阅时，流量值求和，到期时间取最早
