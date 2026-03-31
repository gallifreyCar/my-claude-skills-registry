# Claude Code Skills Registry

[English](README.md) | [中文](README_CN.md)

This repository serves as a comprehensive documentation and registry for all Claude Code skills installed in my environment.

## Purpose

- **Documentation Hub**: Complete reference for all installed skills
- **Usage Guide**: How to use each skill effectively
- **Source Tracking**: Original links and provenance
- **Sync Target**: Auto-updated by `skill-sync` when new skills are installed

## Repository Structure

```
my-claude-skills-registry/
├── README.md                 # This overview file
├── skills/                   # Individual skill documentation
│   ├── built-in/            # Claude Code built-in skills
│   ├── plugins/             # Plugin-based skills
│   └── custom/              # Custom local skills
└── SYNC_LOG.md              # Auto-generated sync history
```

## Skill Categories

### 1. Built-in Skills (Claude Code Native)

| Skill | Description | Trigger | Usage |
|-------|-------------|---------|-------|
| `update-config` | Configure settings.json | Manual | `claude /update-config` |
| `keybindings-help` | Customize keyboard shortcuts | Manual | `claude /keybindings-help` |
| `simplify` | Review code for quality/reuse | Manual | `/simplify` |
| `loop` | Run commands on interval | Manual | `/loop 5m /foo` |
| `schedule` | Create scheduled remote agents | Manual | `/schedule` |
| `claude-api` | Build apps with Claude API | Auto (SDK import) | Imports `anthropic` SDK triggers |

### 2. Plugin Skills

#### web-access (Third-party)
- **GitHub**: https://github.com/eze-is/web-access
- **Description**: All network operations through this skill
- **Triggers**: Search, web fetch, login-required sites, social media scraping
- **Key Features**: CDP browser control, dynamic page rendering, login state preservation

#### claude-mem (Third-party)
- **GitHub**: https://github.com/thedotmack/claude-mem
- **Description**: Persistent cross-session memory system
- **Sub-skills**:
  - `mem-search` - Search past sessions
  - `make-plan` - Create implementation plans
  - `do` - Execute plans
  - `smart-explore` - AST-based code search
  - `timeline-report` - Project history analysis

#### figma (Official Plugin)
- **Source**: claude-plugins-official
- **Sub-skills**:
  - `figma-use` - Figma Plugin API operations
  - `figma-implement-design` - Figma → Code
  - `figma-generate-design` - Code → Figma
  - `figma-generate-library` - Design system library
  - `figma-code-connect` - Code Connect mappings
  - `figma-create-design-system-rules` - Design rules authoring

#### skill-creator (Official Plugin)
- **Source**: claude-plugins-official
- **Description**: Create, test, and optimize skills
- **Features**: Draft skill, run evals, blind comparison, description optimization

### 3. Custom Local Skills

| Skill | Location | Description |
|-------|----------|-------------|
| `feishu-chat-mcp` | ~/.claude/skills/ | Feishu bot chat integration |
| `feishu-doc-mcp` | ~/.claude/skills/ | Feishu document operations |
| `xiaoe-log` | ~/.claude/skills/ | Log query & analysis (TLS/CLS) |

## Installation Locations

- **Custom Skills**: `~/.claude/skills/{skill-name}/`
- **Plugin Cache**: `~/.claude/plugins/cache/{marketplace}/{plugin}/`
- **Settings**: `~/.claude/settings.json`

## Related Repositories

- **skill-sync**: https://github.com/gallifreyCar/skill-sync - Auto-sync tool

## Last Sync

See [SYNC_LOG.md](SYNC_LOG.md) for sync history.