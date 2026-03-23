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

只需配置账号密码，不需要配置 Confluence 地址（从用户提供的链接中自动提取）。

支持以下方式（按优先级）：

1. **Shell 环境变量**（如在 `.zshrc` 中 export）
2. **`.env.local` 文件**（项目根目录）

```
CONFLUENCE_USER=your_username
CONFLUENCE_PASS=your_password
```

## Workflow

### Step 1: Extract Page ID and Base URL

从用户提供的 URL 中提取 pageId 和 base URL：

- URL: `http://confluence.example.com:8090/pages/viewpage.action?pageId=75268574`
  - base URL → `http://confluence.example.com:8090`
  - pageId → `75268574`

提取 base URL 的方式：取 `/pages/` 或 `/rest/` 之前的部分作为 base URL。

### Step 2: Load Credentials

优先使用 shell 环境变量，若未设置则从 `.env.local` 加载：

```bash
if [ -z "$CONFLUENCE_USER" ] || [ -z "$CONFLUENCE_PASS" ]; then
  set -a && source .env.local 2>/dev/null && set +a
fi
```

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
