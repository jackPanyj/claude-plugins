# jb - Claude Code Plugin

读取 Jira issue 和 Confluence 页面的 Claude Code 插件，基于 curl + Basic Auth 直接调用 REST API。

## 安装

```bash
# 1. 添加 marketplace
/plugin marketplace add jackPanyj/claude-plugins

# 2. 安装插件
/plugin install jb@jackPanyj-plugins
```

## Skills

| 命令 | 说明 |
|------|------|
| `/read-jira` | 读取 Jira issue 详情、评论、附件 |
| `/read-confluence` | 读取 Confluence 页面内容，解析 API 文档 |

## 配置

在 `.zshrc` 中 export 或项目根目录 `.env.local` 中配置：

```bash
# Jira
JIRA_USER=your_username
JIRA_PASS=your_password
JIRA_BASE_URL=http://your-jira-host:8080       # 可选，未配置则从链接中提取

# Confluence
CONFLUENCE_USER=your_username
CONFLUENCE_PASS=your_password
CONFLUENCE_BASE_URL=http://your-confluence-host:8090  # 可选，未配置则从链接中提取
```

> 配置了 `JIRA_BASE_URL` / `CONFLUENCE_BASE_URL` 后，可以直接使用 issue key 或 pageId，无需提供完整链接。

## 使用示例

```
# 直接给 issue key
看下 H5-13209

# 给 Jira 链接
看下这个 jira http://your-jira-host/browse/H5-13209

# 给 Confluence 链接
看下这个文档 http://your-confluence-host/pages/viewpage.action?pageId=75268574
```
