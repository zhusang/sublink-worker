## 1. Fetch 层：捕获 subscription-userinfo

- [x] 1.1 修改 `src/parsers/subscription/httpSubscriptionFetcher.js` 的 `fetchSubscription()`：从 response 提取 `subscription-userinfo` 头，在返回值中附带
- [x] 1.2 修改 `fetchSubscriptionWithFormat()`：同样提取并返回 `subscriptionUserinfo` 字段

## 2. Builder 层：聚合 userinfo

- [x] 2.1 在 `src/builders/BaseConfigBuilder.js` 构造函数中初始化 `this.subscriptionUserinfo = null`
- [x] 2.2 新增 `mergeSubscriptionUserinfo(raw)` 方法：解析 userinfo 字符串，与已有值聚合（upload/download/total 求和，expire 取最早）
- [x] 2.3 修改 `parseCustomItems()` 中 HTTP 订阅获取的两处（`fetchSubscription` 和 `fetchSubscriptionWithFormat`），调用 `mergeSubscriptionUserinfo`

## 3. 端点层：设置响应头 + sub_info 参数

- [x] 3.1 所有端点读取 `sub_info` 查询参数
- [x] 3.2 修改 `/singbox` 端点：优先使用 sub_info，否则用 builder.subscriptionUserinfo
- [x] 3.3 修改 `/clash` 端点：同上
- [x] 3.4 修改 `/surge` 端点：移除硬编码假数据，改用 sub_info 或 builder 真实值
- [x] 3.5 修改 `/xray` 端点：在 fetch 循环中收集 userinfo，优先使用 sub_info

## 4. 前端：自定义 subscription-userinfo 输入

- [x] 4.1 `src/i18n/index.js`：添加相关翻译 key（四种语言）
- [x] 4.2 `src/components/Form.jsx`：通用设置区域新增 subscription-userinfo 输入框
- [x] 4.3 `src/components/formLogic.js`：新增 `customSubInfo` 状态 + localStorage 持久化 + submitForm 传 `sub_info` 参数

## 5. 验证

- [x] 5.1 运行 `npm test` 确认无回归
