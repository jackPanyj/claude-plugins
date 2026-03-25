---
name: read-confluence
description: Use when user provides a Confluence link and wants to read API docs or page content. Triggers on "看下这个文档", "读取confluence", "confluence链接", or when user pastes a Confluence URL.
---

# Read Confluence

## Overview

通过 curl + Basic Auth 调用 Confluence REST API，读取页面内容并解析 API 文档。

## When to Use

- 用户提供 Confluence 链接（如 `http://your-confluence-host/pages/viewpage.action?pageId=XXXXX`）
- 用户想查看 API 文档或页面内容

## Prerequisites

支持以下方式（按优先级）：

1. **Shell 环境变量**（如在 `.zshrc` 中 export）
2. **`.env.local` 文件**（项目根目录）

```
CONFLUENCE_USER=your_username
CONFLUENCE_PASS=your_password
CONFLUENCE_BASE_URL=http://confluence.example.com:8090   # optional, 如未配置则从用户提供的链接中提取
```

## Workflow

### Step 1: Extract Page ID and Base URL

**Base URL 获取优先级**：

1. 环境变量 / `.env.local` 中的 `CONFLUENCE_BASE_URL`
2. 从用户提供的 URL 中提取（取 `/pages/` 或 `/rest/` 之前的部分）
3. 若用户只提供了 pageId 且无 `CONFLUENCE_BASE_URL`，则需要用户补充完整链接

示例：

- URL: `http://confluence.example.com:8090/pages/viewpage.action?pageId=75268574`
  - base URL → `http://confluence.example.com:8090`（若未配置 `CONFLUENCE_BASE_URL`）
  - pageId → `75268574`
- 直接提供 pageId：使用 `CONFLUENCE_BASE_URL` 拼接，若未配置则要求用户提供完整链接

### Step 2: Load Credentials

优先使用 shell 环境变量，若未设置则从 `.env.local` 加载：

```bash
if [ -z "$CONFLUENCE_USER" ] || [ -z "$CONFLUENCE_PASS" ]; then
  set -a && source .env.local 2>/dev/null && set +a
fi
```

如果配置了 `CONFLUENCE_BASE_URL`，优先使用它作为 base URL，否则从用户提供的链接中提取。

### Step 3: Fetch Page

用从链接中提取的 base URL 拼接 API 地址：

```bash
curl -s -u "$CONFLUENCE_USER:$CONFLUENCE_PASS" \
  "{baseUrl}/rest/api/content/{pageId}?expand=body.storage,title" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print('TITLE:', d['title']); print('BODY:', d['body']['storage']['value'])"
```

### Step 4: Parse & Present

解析返回的 HTML body，提取关键信息：

#### API 文档格式

| Field          | Where to find                                            |
| -------------- | -------------------------------------------------------- |
| API path (Uri) | Table row where first cell = "Uri"                       |
| Uri params     | Table row where first cell = "Uri参数"                   |
| Method         | Table row where first cell = "Method"                    |
| Description    | Page title or section heading                            |
| Request params | "接口请求参数" table (name, type, required, description) |
| Response       | Code block under "返回结果"                              |

#### 呈现格式

以结构化表格展示：

- **接口列表**: URI、Method、说明
- **请求参数**: 名称、类型、是否必须、描述
- **返回示例**: 格式化的 JSON

## Common Mistakes

- 未检查 shell 环境变量就直接读 `.env.local`
- pageId 提取错误
- HTML body 包含 Confluence storage format 标签，需要解析而非直接展示
