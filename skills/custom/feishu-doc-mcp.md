---
name: feishu-doc-mcp
description: Use when the user wants to read, create, update, or inspect Feishu/Lark cloud docs through Feishu's remote MCP service. Covers Feishu wiki/doc links, permission debugging for TAT-based doc writes, and safe document update workflows.
---

# Feishu Doc MCP

Use this skill when the task involves Feishu/Lark cloud documents and the user wants the agent to read or write them directly.

## What This Skill Does

- Reads Feishu docs through Feishu's remote MCP service
- Creates new docs from Markdown
- Updates existing docs with safer write modes
- Helps diagnose common permission failures for app-based writes

## Required Environment

Before using the bundled script, ensure these env vars are set:

- `FEISHU_APP_ID`
- `FEISHU_APP_SECRET`

The script exchanges them for a Tenant Access Token and then calls Feishu MCP at `https://mcp.feishu.cn/mcp`.

## Default Workflow

Follow this order unless the user explicitly wants a different flow:

1. Read the target doc first with `fetch`.
2. Confirm whether the task is append, targeted replace, or full overwrite.
3. Prefer smaller updates:
   - `append` for adding new content
   - `replace_range` / `insert_before` / `insert_after` for local edits
   - `overwrite` only when intentionally replacing the whole document
4. Re-read the doc after writing if verification matters.

## Safety Rules

- Do not use `overwrite` unless the user clearly wants a full refresh of the document.
- If Feishu returns `forbidden`, stop and report it clearly; do not keep retrying destructive writes.
- If the user provides a wiki/doc URL, pass it directly; do not try to guess tokens manually.
- Treat `replace_all` carefully. If a task can be done with `append` or a targeted replace, prefer that.

## Common Permission Reality

A successful `fetch` does not guarantee `update` will work.

If write calls fail with errors like `CreateDescendant failed: forbidden`, the app usually has one of these problems:

- Missing write permission on the target doc/wiki node
- App not granted access to that doc or knowledge space
- The doc is readable under current scope but not writable by TAT

Report the exact Feishu error back to the user.

## Bundled Script

Use the bundled client:

- Script: `scripts/feishu_mcp_doc.py`

Typical commands:

```bash
python3 scripts/feishu_mcp_doc.py init
python3 scripts/feishu_mcp_doc.py tools
python3 scripts/feishu_mcp_doc.py fetch --doc-id 'https://...'
python3 scripts/feishu_mcp_doc.py create --title 'Title' --markdown '# Body'
python3 scripts/feishu_mcp_doc.py update --doc-id 'https://...' --mode append --markdown '## New Section'
```

## When To Use Which Mode

- `append`: add content to the end
- `replace_range`: replace a located section
- `insert_before` / `insert_after`: insert near a matched section
- `overwrite`: rebuild the whole doc

## If You Need Details

Read the script directly:

- `scripts/feishu_mcp_doc.py`
