# jb - Claude Code Plugin

读取 Jira issue 和 Confluence 页面的 Claude Code 插件，基于 curl + Basic Auth 直接调用 REST API。

## 安装

```bash
/plugin install jb -- source github jackPanyj/claude-plugins
```

## Skills

| 命令 | 说明 |
|------|------|
| `/jb:read-jira` | 读取 Jira issue 详情、评论、附件 |
| `/jb:read-confluence` | 读取 Confluence 页面内容，解析 API 文档 |

## 配置

只需配置账号密码，Jira/Confluence 地址会从你提供的链接中自动提取。

在 `.zshrc` 中 export 或项目根目录 `.env.local` 中配置：

```bash
# Jira
JIRA_USER=your_username
JIRA_PASS=your_password

# Confluence
CONFLUENCE_USER=your_username
CONFLUENCE_PASS=your_password
```

## 使用示例

```
# 直接给 issue key
看下 H5-13209

# 给 Jira 链接
看下这个 jira http://your-jira-host/browse/H5-13209

# 给 Confluence 链接
看下这个文档 http://your-confluence-host/pages/viewpage.action?pageId=75268574
```
