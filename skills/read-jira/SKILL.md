---
name: read-jira
description: Use when user provides a Jira link or issue key (e.g. H5-XXXXX) and wants to read issue details, comments, or understand task requirements. Triggers on "看下这个jira", "读取jira", "jira链接", or when user pastes a Jira URL.
---

# Read Jira

## Overview

通过 curl + Basic Auth 直接调用 Jira REST API，读取 issue 信息和评论。不依赖 MCP。

## When to Use

- 用户提供 Jira 链接（如 `http://your-jira-host/browse/H5-XXXXX`）
- 用户提到 issue key（如 `H5-13209`）想了解详情
- 需要读取 Jira 任务的需求描述或评论

## Prerequisites

只需配置账号密码，不需要配置 Jira 地址（从用户提供的链接中自动提取）。

支持以下方式（按优先级）：

1. **Shell 环境变量**（如在 `.zshrc` 中 export）
2. **`.env.local` 文件**（项目根目录）

```
JIRA_USER=your_username
JIRA_PASS=your_password
```

## Workflow

### Step 1: Extract Issue Key and Base URL

从用户输入中提取 issue key 和 base URL：

- URL: `http://jira.example.com:8080/browse/H5-13209`
  - base URL → `http://jira.example.com:8080`
  - issue key → `H5-13209`
- 直接提供 issue key（如 `H5-13209`）：需要用户补充完整链接，或使用上次使用过的 base URL

提取 base URL 的方式：取 `/browse/` 之前的部分作为 base URL。

### Step 2: Load Credentials

优先使用 shell 环境变量，若未设置则从 `.env.local` 加载：

```bash
if [ -z "$JIRA_USER" ] || [ -z "$JIRA_PASS" ]; then
  set -a && source .env.local 2>/dev/null && set +a
fi
```

### Step 3: Fetch Issue

用从链接中提取的 base URL 拼接 API 地址：

```bash
curl -s -u "$JIRA_USER:$JIRA_PASS" \
  "{baseUrl}/rest/api/2/issue/{issueKey}?fields=summary,status,assignee,reporter,description,comment,priority,issuetype,created,updated,labels,attachment" \
  | python3 -c "
import sys,json
d=json.load(sys.stdin)
f=d['fields']
print('=== Issue:', d['key'], '===')
print('Summary:', f.get('summary'))
print('Type:', f.get('issuetype',{}).get('name'))
print('Status:', f.get('status',{}).get('name'))
print('Priority:', f.get('priority',{}).get('name'))
print('Assignee:', f.get('assignee',{}).get('displayName') if f.get('assignee') else 'None')
print('Reporter:', f.get('reporter',{}).get('displayName') if f.get('reporter') else 'None')
print('Created:', f.get('created','')[:10])
print('Updated:', f.get('updated','')[:10])
labels = f.get('labels',[])
if labels: print('Labels:', ', '.join(labels))
print()
print('--- Description ---')
print(f.get('description') or '(empty)')
print()
comments = f.get('comment',{}).get('comments',[])
print(f'--- Comments ({len(comments)}) ---')
for c in comments:
    print(f'[{c[\"created\"][:10]}] {c[\"author\"][\"displayName\"]}:')
    print(f'  {c[\"body\"]}')
    print()
attachments = f.get('attachment',[])
if attachments:
    print(f'--- Attachments ({len(attachments)}) ---')
    for a in attachments:
        print(f'  - {a[\"filename\"]} ({a[\"size\"]} bytes): {a[\"content\"]}')
"
```

### Step 4: Present Results

将解析后的信息以结构化格式呈现给用户：

- **基本信息**: Summary, Status, Assignee, Priority
- **描述**: 完整的 description 内容
- **评论**: 按时间顺序，包含作者和日期
- **附件**: 文件名和下载链接（如有）

## Common Mistakes

- 未检查 shell 环境变量就直接读 `.env.local`
- URL 中 issue key 大小写错误（Jira key 大小写敏感）
- description 字段使用 Jira wiki markup，不是纯文本，注意格式化展示
