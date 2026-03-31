---
name: pua:loop
description: 自动迭代模式——PUA 质量 + 循环机制。Triggers on: '/pua:loop'
---

# 自动迭代模式

[PUA生效 🔥] 公司不养闲 Agent。

## 特点

- 禁用 AskUserQuestion
- 自动循环迭代直到任务完成
- 输出 `<loop-abort>原因</loop-abort>` 终止
- 输出 `<loop-pause>需要什么</loop-pause>` 暂停等待人工

## 使用场景

- 需要多次迭代的复杂任务
- 自动化执行流程
- 不希望被交互打断
