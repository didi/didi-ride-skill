# 错误处理指南

本文档说明 didi-ride-skill skill 使用过程中可能遇到的错误及解决方案。

> `<skill-dir>` 代表 didi-ride-skill 技能的安装根目录（即 SKILL.md 所在目录），可通过 `openclaw skills info didi-ride-skill` 获取。

## 目录

- [错误处理指南](#错误处理指南)
  - [目录](#目录)
  - [mcporter Missing KEY parameter](#mcporter-missing-key-parameter)
  - [统一错误码表](#统一错误码表)
  - [API 返回 400 错误](#400-错误排查)
  - [常见问题 (FAQ)](#常见问题-faq)
  - [获取帮助](#获取帮助)

***

## mcporter Missing KEY parameter

mcporter 报 `Missing KEY parameter` 时，**不代表 MCP Key 已失效**，禁止直接向用户索要 Key。

可能原因（按概率排序逐一排查）：

1. **`$DIDI_MCP_KEY` 环境变量未展开**：`MCP_URL` 赋值时用了单引号（`'$DIDI_MCP_KEY'`）而非双引号，导致变量字面量传入。确认调用命令中 `MCP_URL` 使用双引号包裹，然后重试。
2. **当前 shell 未注入环境变量**：openclaw 在每次 agent run 启动时自动注入 `DIDI_MCP_KEY`，但手动在终端直接运行 mcporter 时该变量不存在。执行 `echo $DIDI_MCP_KEY` 验证——若为空，在终端手动 `export DIDI_MCP_KEY=<key>` 后重试，或改在 openclaw agent 环境中调用。
3. **mcporter.json 配置异常**：若 `~/.openclaw/workspace/config/mcporter.json` 存在但内容异常，mcporter 可能无法正确构造请求。检查该文件内容是否完整。
4. **Key 本身确实无效**：若以上均排除，执行 `openclaw config get skills.entries.didi-ride-skill.apiKey` 确认 Key 已配置，若返回空则按 `### 3.9 MCP KEY 与配置` 流程重新配置。

***

## 统一错误码表

所有 MCP 工具返回的统一错误码对照：

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| `-32001` | 命中限流 | 请等待一段时间后重试（配额限制按时间窗口重置） |
| `-32002` | 鉴权失败（`auth failed`） | Key 存在但无效或已过期，执行 `### 3.9 MCP KEY 与配置` 的引导流程（含发送二维码） |
| `-32010` | 参数验证失败 | 检查参数格式，确保所有值为字符串 |
| `-32011` | 订单不存在 | 确认订单ID正确 |
| `-32021` | 预估结果过期 | 重新调用价格预估获取新的 traceId |
| `-32030` | 不支持订单类型 | 该类型订单不支持此操作 |
| `-32031` | 订单未支付 | 订单未进入支付状态 |
| `-32040` | 订单已经取消过了 | 订单已被取消，无需重复操作 |
| `-32041` | 订单无法被取消 | 司机已接单或订单已完成，无法通过 API 取消 |
| `-32050` | 内部错误 | 稍后重试，如持续失败请联系客服 |
| `-32060` | 支付失败 | 检查支付账户状态或更换支付方式 |

***

## 400 错误排查

> 💡 **故障排查**：如果配置后 API 返回 400 错误，请检查：
> 1. 参数名是否正确（如 `keywords` 而非 `keyword`）
> 2. 城市名称是否使用完整格式（如 `"北京市"` 而非 `"北京"`）
> 3. 所有参数值是否为字符串格式（加引号）

***

## 常见问题 (FAQ)

**Q: 为什么说"我要上班"没反应？**
A: 需要先配置 `assets/PREFERENCE.md` 中的家和公司地址，以及上班场景的车型偏好。

**Q: 预估价格和实际价格不一致？**
A: 预估价格为参考值，实际费用以行程完成后为准。

**Q: 如何查看历史订单？**
A: 当前 API 仅支持查询 MCP 渠道未完成订单，历史订单请在滴滴 App 中查看。

**Q: 支持哪些城市？**
A: 支持滴滴服务覆盖的所有中国大陆城市。

***

## 获取帮助

如果以上方案无法解决问题，请：

1. 检查 [workflow.md](./workflow.md) 确认操作流程
2. 访问 <https://mcp.didichuxing.com> 获取最新文档
