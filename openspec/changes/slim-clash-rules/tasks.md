## 1. 精简规则定义

- [x] 1.1 修改 `src/config/rules.js` 中的 `UNIFIED_RULES`：移除 Microsoft、Apple、Social Media、Streaming、Gaming、Education、Financial、Cloud Services 共 7 条规则，保留 10 条
- [x] 1.2 修改 `src/config/rules.js` 中的 `PREDEFINED_RULE_SETS.balanced`：从 8 条调整为 `['Location:CN', 'Private', 'Non-China', 'Google', 'Telegram', 'AI Services']`
- [x] 1.3 确认 `PREDEFINED_RULE_SETS.comprehensive` 无需手动修改（它使用 `UNIFIED_RULES.map(rule => rule.name)` 自动派生）

## 2. 验证与测试

- [x] 2.1 运行现有测试 `npm test`，确认无回归失败
- [x] 2.2 检查并更新 `test/selectedRules-compatibility.test.js` 中引用已移除规则名称的用例
- [x] 2.3 检查并更新 `test/subconverter-endpoint.test.js` 中与 comprehensive 预设相关的断言
- [x] 2.4 检查 `test/clash-builder.test.js` 和 `test/worker.test.js` 中的规则数量断言

## 3. 前端确认

- [x] 3.1 检查 `src/components/Form.jsx` 中规则列表的渲染逻辑，确认它从 `UNIFIED_RULES` 动态生成（无硬编码的规则名）
- [x] 3.2 检查 `src/components/formLogic.js` 中是否有硬编码的已移除规则名称引用
