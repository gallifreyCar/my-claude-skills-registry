# Claude Code Skills Registry 中文版

[English](README.md) | [中文](README_CN.md)

本仓库作为 Claude Code 所有已安装技能的综合性文档和注册中心。

## 目的

- **文档中心**：所有已安装技能的完整参考
- **使用指南**：如何有效使用每个技能
- **来源追踪**：原始链接和来源信息
- **同步目标**：安装新技能时由 `skill-sync` 自动更新

## 仓库结构

```
my-claude-skills-registry/
├── README.md                 # 英文概述
├── README_CN.md             # 中文概述
├── skills/                   # 单个技能文档
│   ├── built-in/            # Claude Code 内置技能
│   ├── plugins/             # 插件类技能
│   └── custom/              # 自定义本地技能
└── SYNC_LOG.md              # 自动生成的同步历史
```

## 技能分类

### 1. 内置技能（Claude Code 原生）

| 技能 | 描述 | 触发方式 | 用法 |
|------|------|----------|------|
| `update-config` | 配置 settings.json | 手动 | `claude /update-config` |
| `keybindings-help` | 自定义键盘快捷键 | 手动 | `claude /keybindings-help` |
| `simplify` | 审查代码质量/复用性 | 手动 | `/simplify` |
| `loop` | 定时循环执行命令 | 手动 | `/loop 5m /foo` |
| `schedule` | 创建定时远程代理 | 手动 | `/schedule` |
| `claude-api` | 使用 Claude API 构建应用 | 自动（SDK 导入） | 导入 `anthropic` SDK 时触发 |

### 2. 插件技能

#### web-access（第三方）
- **GitHub**: https://github.com/eze-is/web-access
- **描述**：所有网络操作通过此技能处理
- **触发场景**：搜索、网页抓取、需要登录的网站、社交媒体抓取
- **核心功能**：CDP 浏览器控制、动态页面渲染、登录状态保持

#### claude-mem（第三方）
- **GitHub**: https://github.com/thedotmack/claude-mem
- **描述**：跨会话持久化记忆系统
- **子技能**：
  - `mem-search` - 搜索历史会话
  - `make-plan` - 创建实施计划
  - `do` - 执行计划
  - `smart-explore` - 基于 AST 的代码搜索
  - `timeline-report` - 项目历史分析

#### figma（官方插件）
- **来源**：claude-plugins-official
- **子技能**：
  - `figma-use` - Figma Plugin API 操作
  - `figma-implement-design` - Figma → 代码
  - `figma-generate-design` - 代码 → Figma
  - `figma-generate-library` - 设计系统库
  - `figma-code-connect` - Code Connect 映射
  - `figma-create-design-system-rules` - 设计规则编写

#### skill-creator（官方插件）
- **来源**：claude-plugins-official
- **描述**：创建、测试和优化技能
- **功能**：起草技能、运行评测、盲测对比、描述优化

### 3. 自定义本地技能

| 技能 | 位置 | 描述 |
|------|------|------|
| `feishu-chat-mcp` | ~/.claude/skills/ | 飞书机器人聊天集成 |
| `feishu-doc-mcp` | ~/.claude/skills/ | 飞书文档操作 |
| `xiaoe-log` | ~/.claude/skills/ | 日志查询与分析（TLS/CLS） |

## 安装位置

- **自定义技能**：`~/.claude/skills/{skill-name}/`
- **插件缓存**：`~/.claude/plugins/cache/{marketplace}/{plugin}/`
- **配置文件**：`~/.claude/settings.json`

## 相关仓库

- **skill-sync**：https://github.com/gallifreyCar/skill-sync - 自动同步工具

## 最近同步

查看 [SYNC_LOG.md](SYNC_LOG.md) 了解同步历史。