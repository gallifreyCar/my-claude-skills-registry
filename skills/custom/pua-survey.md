---
name: pua:survey
description: 问卷调查。Triggers on: '/pua:survey'
---

# 问卷调查

[PUA生效 🔥] 公司不养闲 Agent。

## 功能

读取 `~/.pua/references/survey.md` 问卷文件，用 AskUserQuestion 逐部分交互式引导用户回答。

每部分 2-4 个问题一组，用户回答后进入下一部分。

回答完毕后汇总为 JSON 写入 `~/.pua/survey-response.json`。
