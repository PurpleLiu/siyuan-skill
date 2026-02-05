---
name: siyuan-mcp
description: SiYuan Note MCP tools for searching and managing notebooks, documents, blocks, and exports.
homepage: https://github.com/PurpleLiu/siyuan-mcp
metadata:
  {
    "openclaw":
      {
        "emoji": "📔",
        "requires": { "env": ["SIYUAN_TOKEN", "SIYUAN_BASE_URL"] },
        "primaryEnv": "SIYUAN_TOKEN"
      }
  }
---

# siyuan-mcp

Use the SiYuan MCP server to search, read, create, and update documents and notebooks—similar structure and workflows to Notion.

## Setup

1. Get your SiYuan API token from Settings.
2. Set the base URL for your SiYuan instance (local or remote).
3. Export the environment variables before using mcporter:

```bash
export SIYUAN_TOKEN="your_token_here"
export SIYUAN_BASE_URL="http://127.0.0.1:6806"
```

4. Ensure mcporter has a configured server named `siyuan` pointing to the PurpleLiu/siyuan-mcp server:

```bash
mcporter config add siyuan \
  --command "siyuan-mcp stdio --token $SIYUAN_TOKEN --baseUrl $SIYUAN_BASE_URL"
```

## Using mcporter

Call tools with:

```bash
mcporter call siyuan.<tool_name> key=value
```

List all available tools and params:

```bash
mcporter list siyuan --all-parameters
```

## Common Tool Examples

**Search (unified_search):**

```bash
mcporter call siyuan.unified_search content="project roadmap" limit=5
```

**Get document content:**

```bash
mcporter call siyuan.get_document_content document_id="20250108185901-y0f6g29"
```

**Create document:**

```bash
mcporter call siyuan.create_document \
  notebook_id="20250108185901-y0f6g29" \
  path="/Meeting Notes" \
  content="# Notes\n- Item 1"
```

**Append to daily note:**

```bash
mcporter call siyuan.append_to_daily_note \
  notebook_id="20250108185901-y0f6g29" \
  content="- Follow up with vendor"
```

**List notebooks:**

```bash
mcporter call siyuan.list_notebooks
```

**Create snapshot:**

```bash
mcporter call siyuan.create_snapshot memo="before-refactor"
```

## Notes

- Tooling is provided by the PurpleLiu/siyuan-mcp server.
- SiYuan docs: https://github.com/siyuan-note/siyuan and https://ld246.com (official community).
- Tool names and parameters come from the MCP server schema; list them via mcporter if unsure.
