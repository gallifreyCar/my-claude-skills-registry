---
name: xiaoe-log
description: 使用 meta-mcp 工具查询并分析日志。支持国内（火山云 TLS / 腾讯云 CLS）和海外（腾讯云 CLS）区域。遇到错误排查、trace_id 追踪、超时/异常分析或日志统计请求时调用。
---

# 日志查询（国内 TLS & CLS / 海外 CLS）

## 核心要求
- 使用 `meta-mcp_get_log_topics`、`meta-mcp_search_log` 和 `meta-mcp_get_service_info` 三个 MCP 工具
- 所有工具支持 `region` 参数：
  - `domestic`（国内火山云 TLS，默认）
  - `domestic-cls`（国内腾讯云 CLS）
  - `overseas`（海外腾讯云 CLS）
- 严禁使用 `functions.` 或 `toolu_` 前缀
- 每次调用工具后等待返回结果再继续

## ⚠️ 强制交互协议（不可违反）
1. **禁止一次性完成**：严禁在同一个 Thinking 块中既提出查询方案又调用 `meta-mcp_search_log`。
2. **确认锁机制**：在用户针对”参数摘要”回复”确认”、”yes”或”开始”之前，**禁止**调用 `meta-mcp_search_log`。`meta-mcp_get_log_topics` 和 `meta-mcp_get_service_info` 属于预检操作，在第一阶段自动执行，不受确认锁约束。
3. **状态检查**：在发起 `meta-mcp_search_log` 调用前，必须在 Thinking 中自检：”用户是否已经对当前的查询参数（Query、时间范围）表示了明确认可？”

## ⚠️ 环境强制约束
- **MCP 工具禁止 Bash 调用**：禁止使用 `npx`、`echo |`、`python` 或任何终端命令来运行 `meta-mcp_*` 工具，MCP 工具只能通过模型原生的 `tool_use` 接口调用。
- **Bash 允许范围**：允许使用 Bash 执行辅助操作（如 `git remote get-url origin` 获取仓库地址）。
- **时间获取强制使用 Python**：所有时间戳和可读时间的获取必须使用 Python，禁止使用 Bash `date` 命令（Windows Git Bash 的 TZ 环境变量不可靠）。
- **报错即停止**：如果底层 MCP 报错（如 SSE error），请直接向用户报错”MCP 链路中断”，严禁尝试通过 Bash 编写补救脚本。

## 快速流程
一. 第一阶段：方案设计与预检（必须在此停顿，只需用户回复一次）
1. 解析请求：服务名、region（国内/海外）、**意图类型**、时间范围、关键字/trace_id、分析目标、**输出级别**

2. **意图类型判断**（用于选择正确的 log topic）：
   - **网关类型**：用户请求中包含 "gateway"、"网关"、"small-gateway" 等关键词 → 查询 small-gateway-log
   - **任务类型**：用户请求中包含 "task"、"任务"、"job"、"worker" 等关键词 → 查询 task-log
   - **业务类型**：用户请求中包含 "web"、"service"、"业务"、"api" 等关键词 → 查询 service-log
   - **未明确**：默认查询 service-log
   - 用户明确指定 topic 时以用户为准

3. **输出级别推断**：根据用户意图的深度和紧迫度自主判断，不依赖固定关键词
   - **精简**：用户只需快速确认是否有问题（巡检、概览、确认状态）
   - **标准**：用户需要排查定位问题（日常排查、查看具体错误、了解影响范围）
   - **详细**：用户需要深入分析原因或产出报告（根因分析、复盘、汇报、技术调查）
   - 用户明确指定级别时以用户为准
   - 判断不确定时默认标准
3. 服务名解析（详见 [服务名解析规则](#服务名解析)）：
   - 用户传入服务名 → **直接使用，但需要尝试分隔符变体**
   - 用户未传入 → Bash 获取 git remote URL → 调用 `meta-mcp_get_service_info` 从 CMDB 获取精确服务名（`sys_en_name`）；若 CMDB 未注册则降级为仓库名推测
4. **并行执行**（以下操作同时发起，不串行等待）：
   - a. 预检 topic_id：**从用户请求中识别 region（国内/海外），并行尝试服务名的分隔符变体**
     - 原始名（如 `h5-gateway`）
     - 分隔符变体：`-` ↔ `_`（如 `h5_gateway`）
     - 任一命中即使用该结果；都未命中则报错
   - b. 获取时间：通过 Python 一次性获取当前时间戳和可读时间（禁止 AI 心算或编造时间）
   - c. **意图分析**：根据用户请求中的关键词判断意图类型（gateway / task / service），确定要查询的 log 类型

   **Region 识别规则**：
   - 用户说"海外"、"overseas"、"国外" → `region="overseas"`
   - 用户说"国内 CLS"、"Tencent"、"腾讯云"（国内场景） → `region="domestic-cls"`
   - 用户说"国内"、"domestic"、"TLS"、"火山云" → `region="domestic"`
   - 用户说"elink"或其变体（eLink、ELINK、e-link 等） → `region="overseas"`（海外环境）
   - 用户未指定 → `region=""`（使用默认值 domestic）
5. 生成查询：构建 Lucene 语句，默认全文检索
6. 展示一站式参数摘要并等待用户确认（回复 "yes" 执行），摘要中包含：
   - 查询参数（服务、Query、限制条数）
   - **Region**：识别出的区域（domestic/domestic-cls/overseas），显示在摘要中
   - **意图类型**：网关 / 任务 / 业务（gateway / task / service），以及选择的 log 类型
   - topic 验证状态（已通过预检）
   - 时区（首次查询标注默认值，后续沿用已确认时区）
   - 时间范围：用户已指定 → 直接使用；用户未指定 → 默认"近 15 分钟"并在摘要中列出可选项（近30分钟/1小时/3小时/6小时），用户可在同一次回复中修改
   - 时间戳及对应的可读时间
   - **输出级别**：展示推断结果及可选项（精简/标准/详细）

二. 第二阶段：自动化执行（获得用户"yes"后触发）
7. 使用已缓存的 topic_id 和已计算的时间戳直接调用 `meta-mcp_search_log`
8. 生成分析报告（三步，不可跳过）：
   a. **确认输出模板**：如果当前上下文中尚未包含 `references/output-format.md` 的内容，则读取该文件；如果已在本次会话中读取过，可跳过重复读取
   b. **按输出级别生成报告**（模板是参考，可根据日志内容和用户意图灵活调整）：
      - **精简模式**：📊一行统计 + 📈级别分布 + ⚠️关键发现1条 + 🔗Trace ID 1个
      - **标准模式**：📊一行统计 + 📈级别分布 + 🏷️错误Top3 + ⚠️关键发现2-3条 + 💡建议1-2条
      - **详细模式**：完整模板（含典型样本、关联分析、三级建议等）
      - Agent 可根据实际日志内容和用户上下文灵活调整各区块的展示深度和内容
   c. **文件路径检查**：如果本次查询过程中创建了本地日志文件，必须在报告末尾追加 `📁 日志文件: {path} | 大小: {size}`。遗漏文件路径视为任务未完成

## 服务名解析
- **用户传入服务名**：直接使用
- **用户未传入服务名**：通过 Bash `git remote get-url origin` 获取仓库地址 → 转 SSH 格式 → 调用 `meta-mcp_get_service_info` 从 CMDB 获取精确 `sys_en_name`；若 CMDB 未注册则降级为仓库名推测，并在参数摘要中标注"（推测，可能不准确）"
- **topic_id 预检兜底（用户传入服务名场景）**：并行发出原始名 + 分隔符变体（`-` ↔ `_`）两个 `get_log_topics` 请求，任一命中即可 → 都未命中 → 通过仓库地址调用 `get_service_info` 反查 CMDB 服务名 → 纠正后的服务名直接体现在参数摘要中，用户一次确认即可
- **详细规则**：参见 [服务名解析](references/service-resolution.md)

## 查询策略
- 默认全文检索：`ERROR`、`*timeout*`、`*exception*`、`*trace片段*`
- 备用字段检索：`level`、`message`、`trace_id`（需键值索引）
- 结合预留字段过滤：`__container_name__`、`__namespace__` 等

### 空结果重试策略
查询无结果时，按以下顺序放宽条件（最多重试 2 次）：
1. **去掉 level 过滤**：`level:ERROR` → `ERROR`（改为全文检索）
2. **扩大时间范围**：15min → 1h
3. **简化关键词**：去掉通配符，缩短关键词
4. 仍无结果则告知用户，建议调整查询条件

### 多 Topic 查询策略
`get_log_topics` 返回多个 topic_id 时：
- 根据意图类型选择匹配的 topic：
  - **网关类型**：优先选择包含 `small-gateway`、`gateway` 等关键词的 topic
  - **任务类型**：优先选择包含 `task`、`job`、`worker` 等关键词的 topic
  - **业务类型**：优先选择包含 `service`、`api` 等关键词的 topic
  - **未明确意图**：查询所有 topic，合并结果后统一分析
- 在报告中标注实际查询的 topic 和日志数量

## 时间范围与时区
- **核心原则**：永远通过 Python 获取实际时间戳和可读时间（13 位毫秒整数），禁止 AI 心算，禁止使用 `0`
- **用户指定时间范围**：使用 Python 获取当前时间戳后计算对应的 `start_time` 和 `end_time`
- **用户未指定时间范围**：在参数摘要中提供常用时间范围选项（近15分钟/30分钟/1小时/3小时/6小时），默认推荐近 15 分钟
- **时区**：首次查询在参数摘要中标注默认时区 `Asia/Shanghai (UTC+8)` 供用户确认；后续查询沿用已确认时区
- **时区转换**：使用 Python 的 `timezone(timedelta(hours=N))` 进行时区转换，确保准确性
- **详细规则**：参见 [时间处理](references/time-handling.md)

## 引用使用指引
- 需要明确执行顺序与强制流程时查 [工作流程](references/workflow.md)
- 需要时间范围确认、时间戳计算与时区规则时查 [时间处理](references/time-handling.md)
- 需要参数摘要规则与示例时查 [参数摘要](references/parameter-summary.md)
- 需要服务名解析与 topic_id 兜底时查 [服务名解析](references/service-resolution.md)
- 需要查询语法、索引约束、场景示例时查 [查询指南](references/query-guide.md)
- **需要输出结构时必须使用** [输出格式](references/output-format.md) **中的标准模板**
- 需要特定场景分析模式时参考 [分析模式](references/log-analysis-patterns.md)
- 需要 MCP 工具参数与响应细节时查 [API 参考](references/api-reference.md)

## 重要约束
- **日志文件仅作信息展示**：当日志保存到本地时，仅在输出末尾展示文件路径和大小
- **所有查询均为新查询**：通过精确的查询条件获取所需日志

## ⚠️ 输出格式强制约束（末尾锚点，不可忽略）
- 禁止凭记忆生成输出格式。首次生成报告前必须读取 `references/output-format.md`；同一会话内重复查询可复用已读取的模板
- **按输出级别匹配必需区块**：
  - **精简模式**：📊一行统计、📈级别分布、⚠️关键发现1条、🔗Trace ID 1个
  - **标准模式**：📊一行统计、📈级别分布、🏷️错误Top3、⚠️关键发现2-3条、💡建议1-2条
  - **详细模式**：完整区块（含典型样本、关联分析、三级建议等）
- 如果本次查询保存了本地日志文件，报告末尾必须包含 📁 文件路径和大小，遗漏视为任务失败
