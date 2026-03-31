---
name: didi-ride-skill
description: 中国城市出行服务。当用户表达任何交通出行需求时必须使用此技能——包括打车/叫车/网约车、查价格、路线规划（公交/驾车/步行/骑行）、周边搜索、查询订单/司机位置/取消订单。关键词："打车"、"叫车"、"去[地点]"、"回家"、"上班"、"下班"、"查价格"、"多少钱"、"路线"、"怎么走"、"步行到"、"附近"、"周边"、"司机"、"订单"、"查询订单"。注意：即使用户未明确说"打车"，只要涉及从A地到B地、通勤、或交通方式选择，都应触发。不触发场景：开发打车应用、使用其他导航app、订外卖、查公交时刻表、股票/财报查询。
homepage: https://mcp.didichuxing.com
metadata:
  { "openclaw": { "emoji": "🚕", "always": true, "requires": { "bins": ["openclaw", "mcporter"], "env": ["DIDI_MCP_KEY"] }, "primaryEnv": "DIDI_MCP_KEY", "install": [{ "id": "node", "kind": "node", "package": "mcporter", "bins": ["mcporter"], "label": "Install mcporter (node)" }] } }
---

# 滴滴出行服务 (DiDi Ride Skill)

通过 DiDi MCP Server API 提供打车、查询订单、司机位置、预约叫车、路线规划、周边搜索能力。

---

## 1. 快速开始（2 分钟）

### 1.1 获取 MCP KEY

**方式一：用「滴滴出行App」扫码（推荐，最快）**

![滴滴出行APP扫码获取MCP Key，解锁一键打车](https://s3-yspu-cdn.didistatic.com/mcp-web/qrcode/didi_ride_skill_qrcode.png)
> ⚠️ **Agent 注意**：用户客户端无法渲染 Markdown 图片，**禁止直接输出上方图片语法**。需向用户发送二维码时，执行 `### 3.9 MCP KEY 与配置` 中的 `openclaw message send` 命令发图。

打开滴滴出行 App，扫描二维码，即可快速获取 MCP Key。

**方式二：访问官网**

访问 https://mcp.didichuxing.com/claw 获取您的 MCP Key。

### 1.2 配置 Key

**方式一：对话中输入（推荐）**

直接在对话中告诉我您的 MCP Key，我会帮您配置：

```
你: 我的 MCP Key 是 xxxxxx
```

**方式二：OpenClaw 配置文件**

编辑 `~/.openclaw/openclaw.json`，添加：

```json
{
  "skills": {
    "entries": {
      "didi-ride-skill": {
        "enabled": true,
        "apiKey": "你的MCP_KEY"  // apiKey 是 OpenClaw 标准字段名，存储的值就是滴滴平台的 MCP KEY
      }
    }
  }
}
```

### 1.3 开始使用

配置完成后，直接对话即可：

```
你: 打车去北京西站
你: 帮我查一下从国贸到三里屯的路线
你: 查询订单
```

首次使用时，OpenClaw 会提示安装 mcporter 工具。

---

## 2. 用户指南

本 Skill 支持以下操作：

- **打车**：直接说"打车去[地点]"、"回家"、"上班"
- **查价**：查一下从 A 到 B 多少钱
- **查询订单**：输入「查询订单」了解当前订单状态（司机位置、行程进度等）
- **司机位置**：司机在哪里、多久到
- **预约出行**："15分钟后打个车"、"明天9点去机场"
- **路线规划**：驾车/公交/步行/骑行路线
- **取消订单**：取消当前订单

---

## 3. Agent 执行指令

以下内容为 AI 执行参考，用户可忽略。

### 3.1 文件地图 

按需读取以下文件，不要猜测未读过的内容：

| 文件 | 用途 | 何时读取 |
|------|------|----------|
| `SKILL.md` | 触发、主流程、硬性门禁、查询订单规则、预约出行规则 | 每次触发必读 |
| `references/workflow.md` | 分阶段详细流程与命令范式 | 需要实现细节时读 |
| `references/api_references.md` | MCP 函数签名与参数定义 | 每次调用工具前**必须**核对 |
| `references/error_handling.md` | 常见错误与恢复策略 | ⚠️ 遇到调用失败时（比如 400 错误）必须读取此文件 |
| `references/setup.md` | 安装 mcporter、配置 MCP KEY 的完整步骤 | 用户询问安装/配置问题时读 |
| `assets/PREFERENCE.md` | 地址别名/车型/手机号偏好 | 用户提到别名地址（家、公司、妈妈家等）、车型、手机号，或未明确给出起终点时**必须**读取。别名匹配规则见执行前检查第 6 条 |

### 3.2 执行前检查

1. **检查 mcporter**：若 `mcporter` 不存在（`command not found`），停止并引导用户阅读 `references/setup.md`。没有 mcporter 就无法调用任何 MCP 工具，后续任何流程都无法执行。

2. **检查 Key**：执行 `openclaw config get skills.entries.didi-ride-skill.apiKey`，若输出为空或非 `__OPENCLAW_REDACTED__`，按 `### 3.9 MCP KEY 与配置` 流程引导。Key 缺失时 mcporter 的报错信息具有误导性，不要尝试绕过。

3. **mcporter 调用格式**：

```bash
MCP_URL=”https://mcp.didichuxing.com/mcp-servers?key=$DIDI_MCP_KEY”
mcporter call “$MCP_URL” <tool> --args '{“key”:”value”}'
```

4. **参数值必须加引号**（字符串格式），否则 API 会报”缺少必填参数”。
5. **先预估再下单**：`taxi_create_order` 依赖 `taxi_estimate` 返回的 `traceId`，没有 traceId 下单会失败。traceId 有时效性，过期（`-32021` 错误）需重新预估。
6. **起终点处理**：
   - 坐标必须来自 `maps_textsearch`，不要凭空猜测坐标。
   - **禁止用对话历史记忆补充起终点**——用户可能已经换了地方。
   - **起终点缺失时**按以下顺序补全：
     - ① 读 `assets/PREFERENCE.md`，若有地址别名**且地址值非空**，根据场景推断（如早晨→起点"家"、下班→起点"公司"）。别名行存在但地址为空 = 未配置。
     - ② 若无可用别名，直接询问用户。
   - **别名匹配规则（精确优先）**："家"只匹配别名"家"，不匹配"妈妈家"；需明确含"妈妈"语义才匹配"妈妈家"。其他自定义别名同理。
   - **推断的起终点、或 `maps_textsearch` 返回多个候选结果时，必须在主流程 step 2 向用户确认**；用户明确指定且精确匹配的地点无需确认。

### 3.3 用户确认策略

| 场景 | 规则 |
|------|------|
| 打车（实时/预约） | 推断的地址或搜索返回多个候选时必须确认起终点（见主流程 step 2），用户明确指定且精确匹配时无需确认，确认后再预估下单 |
| 取消订单 | 即使用户说了"取消订单"，仍必须先明确询问"确认取消吗？"，等用户回复确认后才能调用 `taxi_cancel_order`。用户的取消意图 ≠ 取消确认。 |

### 3.4 主流程（最小可执行）

1. 地址解析：`maps_textsearch`（必要时结合 `assets/PREFERENCE.md`，按执行前检查第 6 条处理）。
2. 确认起终点：推断的地址（如别名推断）或 `maps_textsearch` 返回多个候选结果时，必须向用户确认（如"从家出发去首都机场T3航站楼，确认吗？"）；用户明确指定且精确匹配的地点无需确认。用户纠正则按纠正内容重新解析。
3. 价格预估：`taxi_estimate`，记录 `traceId`。
4. 车型决策：
   - 用户明确车型：直接用；
   - 用户未明确：使用 `assets/PREFERENCE.md` 中的场景车型偏好或默认车型；
   - 偏好也未配置：向用户询问车型，不要自行推荐。
   - 可用车型以 `taxi_estimate` API 返回为准，获取对应的 `productCategory` 创建订单。
5. 创建订单：`taxi_create_order`（使用最新 `traceId`）。
6. 结果输出：给出订单号、起终点、车型、预估价，末尾提示 `💡 发送「查询订单」可了解当前订单状态`。
7. ⚠️ 自动回查（必做）：根据 `### 3.8 发单后自动回查` 中的 cron 指令内容，创建定时任务，参数信息严格遵循章节内要求。此步不可省略。

### 3.5 偏好设置更新

当用户要求设置/记住/记一下/帮我记/保存地址别名、车型偏好或手机号时，**必须**直接编辑 `assets/PREFERENCE.md` 对应的 markdown 表格行。

- **地址别名**（"我家在…"、"公司在…"、"儿子的学校是…"、"妈妈家在…"）：先调用 `maps_textsearch` 解析地址获取坐标，然后更新「地址别名」表——已有别名更新对应行，新别名追加新行。别名由用户定义，不限于"家""公司"。
- **场景车型**（"上班用快车"、"下班用特惠和快车"）：更新「场景车型偏好」表对应行。品类代码参考表底注释，多车型用英文逗号分隔（如 `1,201`）。
- **叫车手机号**（"我的手机号是…"）：更新「默认偏好」表中的叫车手机号行。
- **创建订单时**：若 PREFERENCE.md 中配置了叫车手机号，将其作为 `caller_car_phone` 参数传入 `taxi_create_order`。

### 3.6 查询订单

触发词：`查询订单` / `查询订单 <orderId>`

订单号来源（优先级从高到低）：
1. 用户消息中明确给出；
2. 当前对话上下文中最近一次创建的订单号；
3. 以上均无时，向用户询问。

调用命令：

```bash
MCP_URL="https://mcp.didichuxing.com/mcp-servers?key=$DIDI_MCP_KEY"
mcporter call "$MCP_URL" taxi_query_order --args '{"order_id":"ORDER_ID"}'
```

#### 3.6.1 状态码与输出规则

| code | 含义 | 必须输出 |
|------|------|----------|
| 0 | 匹配中 | ⏳ 正在为您匹配司机，请稍候 |
| 1 | 司机已接单 | **必须展示**：司机姓名、车型、车牌、电话；距上车点距离和预计到达时间 |
| 2 | 司机已到达 | 🔔 司机已到达上车点，请前往上车 |
| 4 | 行程进行中 | 🚗 行程已开始 |
| 5 | 订单完成 | ✅ 行程结束，展示费用（如有） |
| 6 | 订单已被系统取消 | ❌ 订单已被系统取消 |
| 7 | 订单已被取消 | ❌ 订单已取消 |
| 3/8-12 | 其他终态 | 显示对应状态描述 |

### 3.7 预约出行规则

当用户要求在特定时间叫车（如"15分钟后"、"明天9点"）：

- 使用 cron 一次性任务（`--at`），到点由 isolated agent 独立执行完整打车流程；
- ⚠️ `--message` 必须包含完整起终点（带城市前缀）和车型，isolated session 无历史上下文。起点或终点缺失时先从 `assets/PREFERENCE.md` 推断并向用户确认，推断不了则必须先询问用户，**不要自行推断未知信息**；
- 到点后 agent 自行执行：地址解析 → 价格预估（获取最新 traceId）→ 创建订单；
- 如果无法获取当前会话的 metadata，不要创建 cron，改为在回复中提醒用户主动查询；
- 如果定时任务创建失败，必须在回复中提示用户，不能默默失败。

```bash
# ⚠️ 替换占位符：
#   FROM_NAME    → 带城市前缀的起点全称（如"北京市西二旗地铁站"）
#   TO_NAME      → 带城市前缀的终点全称（如"北京市佰嘉城小区"）
#   VEHICLE      → 车型（如"快车"）
#   TIME         → 见下方时间规则
#   CHANNEL_NAME → 当前会话 metadata 中的 channel 字段（如 feishu、telegram），CHANNEL_NAME 不需要带引号，例如: feishu ✅, "feishu" ❌。不允许使用 last 作为参数值。
#   CHAT_ID      → 当前会话 metadata 中的 chat_id 字段


openclaw cron add \
  --name "didi-ride-skill:$(date +%s)" \
  --at "TIME" \
  --session isolated \
  --message "执行定时打车：起点「FROM_NAME」，终点「TO_NAME」，车型「VEHICLE」。请完整执行打车流程：地址解析 → 价格预估（获取最新 traceId）→ 创建订单。订单创建成功后，输出订单信息并提示用户可发送「查询订单」了解订单状态，同时创建 5 分钟后自动回查 cron（模板见 SKILL.md 第 3.8 节「发单后自动回查」）。" \
  --announce \
  --channel CHANNEL_NAME \
  --to "CHAT_ID"
```

#### 3.7.1 TIME 填写规则

| 场景 | 写法 | 示例 |
|------|------|------|
| 相对时间（X 分钟/小时后） | duration 格式 | `15m` / `2h` / `1h30m` |
| 绝对时间（具体时刻） | 本地时区 ISO 格式 | `$(date -d '明天 09:00' '+%Y-%m-%dT%H:%M:%S+08:00')` |

- 相对时间（如 `15m`）无需格式化，直接使用
- 绝对时间使用带时区的 ISO 8601 格式：`YYYY-MM-DDTHH:MM:SS+08:00`（北京时间东八区）

**系统兼容性说明：**
- Linux (GNU date): `date -d '明天 09:00' '+%Y-%m-%dT%H:%M:%S+08:00'`
- macOS (BSD date): `TZ=Asia/Shanghai date -j -v+1d -f '%H:%M' '09:00' '+%Y-%m-%dT%H:%M:%S+08:00'`

### 3.8 发单后自动回查

发单成功后（实时单或预约单），根据本章节下文 "openclaw cron add" 命令，创建 cron 任务，务必按照本章节下文bash命令参数设置。
- 如果无法获取当前会话的 metadata，不要创建 cron，改为在回复中提醒用户主动查询订单状态。
- 如果定时任务创建失败，必须在回复中提示用户，不能默默失败。

```bash
# ⚠️ 替换占位符：
#   ORDER_ID     → 实际订单号（taxi_create_order 返回）
#   CHANNEL_NAME → 当前会话 metadata 中的 channel 字段（如 feishu、telegram），CHANNEL_NAME 不需要带引号，例如: feishu ✅, "feishu" ❌。不允许使用 last 作为参数值。
#   CHAT_ID      → 当前会话 metadata 中的 chat_id 字段

openclaw cron add \
  --name "didi-ride-skill:check:ORDER_ID" \
  --at "5m" \
  --session isolated \
  --message "查询滴滴订单状态：订单号 ORDER_ID。调用 taxi_query_order 查询并输出当前状态。如果司机已接单，输出司机姓名、车型、车牌、电话及预计到达时间；如果仍在匹配中，提示用户耐心等待。" \
  --announce \
  --channel CHANNEL_NAME \
  --to "CHAT_ID"
```

### 3.9 MCP KEY 与配置

> **术语说明**：滴滴平台称此凭证为「MCP KEY」，OpenClaw 配置字段统一叫 `apiKey`，注入后的环境变量为 `DIDI_MCP_KEY`——三者是同一个值。

⚠️ Key 来源：通过 `openclaw config set` 持久化后，OpenClaw 在每次 agent run 启动时自动注入为环境变量 `DIDI_MCP_KEY`。
```bash
# ⚠️ 可以通过以下命令判断当前 DIDI_MCP_KEY 是否配置。如果输出 __OPENCLAW_REDACTED__ ，即为已配置，可在后面使用环境变量`DIDI_MCP_KEY`。
openclaw config get skills.entries.didi-ride-skill.apiKey
```

#### 3.9.1 用户在对话中提供 Key 时（必须持久化）

当用户回复了 Key（如"我的 Key 是 xxxxxx"），**必须**执行以下命令持久化，**严禁只 export 到当前 shell**：

```bash
# ⚠️ 将 YOUR_KEY 替换为用户提供的实际值
openclaw config set 'skills.entries.didi-ride-skill.apiKey' 'YOUR_KEY'
```

持久化成功后：
- OpenClaw 在所有后续 agent run（含 cron isolated session）中自动注入 `DIDI_MCP_KEY`
- 配置更改后会自动生效，无需额外操作指令。

⚠️ Key 未配置**或** MCP 返回鉴权失败（`error.code: -32002`）时，执行以下两步：

1. 执行下面的 "openclaw message send" 命令，发送二维码图片（`{CHAT_ID}` → metadata 的 chat_id，`{CHANNEL_NAME}` → metadata 的 channel）：

```bash
openclaw message send --channel {CHANNEL_NAME} --target {CHAT_ID} --media "https://s3-yspu-cdn.didistatic.com/mcp-web/qrcode/didi_ride_skill_qrcode.png" --message "滴滴出行APP扫码获取MCP Key，解锁一键打车"
```

2. 输出文字：

> 您还没有配置 DIDI_MCP_KEY 或 Key 已失效，请访问 [滴滴MCP平台](https://mcp.didichuxing.com/claw) 获取 MCP KEY，然后配置环境变量或在 OpenClaw 配置文件中设置。

### 3.10 工具清单

| 领域 | 工具 |
|------|------|
| 打车 | `maps_textsearch`, `taxi_estimate`, `taxi_create_order`, `taxi_query_order`, `taxi_get_driver_location`, `taxi_cancel_order`, `maps_regeocode`, `taxi_generate_ride_app_link`（用户无 API 直发权限时的备选：生成深度链接让用户在 App 内完成发单） |
| 路线 | `maps_direction_driving`, `maps_direction_transit`, `maps_direction_walking`, `maps_direction_bicycling` |
| 周边 | `maps_place_around` |
