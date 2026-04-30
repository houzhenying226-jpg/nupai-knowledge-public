# zsh heredoc 截断 markdown / 正则内容

> 类型：故障复盘 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw 部署期间，多次出现 Python heredoc 写文件内容被截断的问题。表现：写入的 markdown 表格不完整、正则字符串被提前关闭、bold 语法（text）导致 heredoc 提前结束。

## 正文

## 背景

OpenClaw 部署期间，多次出现 Python heredoc 写文件内容被截断的问题。表现：写入的 markdown 表格不完整、正则字符串被提前关闭、bold 语法（**text**）导致 heredoc 提前结束。

## 根因分析

zsh 在处理 heredoc 时会对某些特殊字符进行解析，当 Python 代码中的字符串内容包含以下元素时，zsh 可能在 Python 执行 heredoc 之前就进行了错误解析：
- markdown 表格中的 `|` 字符
- 正则表达式中的 `*`、`+`、`?`
- bold 语法 `**text**`（与 zsh glob 冲突）
- bash 变量引用 `$var`（在非单引号 heredoc 中展开）

## 处理方式

**彻底解决方案：不用 heredoc，改用 Write tool 直接写文件。**

当必须用 bash 写文件时，使用单引号 heredoc：
```bash
cat << 'EOF' > file.py
# 内容中的 $var、**bold**、正则 都不会被解析
EOF
```

Claude Code 中推荐始终用 Write tool，不依赖 shell heredoc 写代码/markdown 文件。

## 经验教训

- Write tool 写文件比 bash heredoc 更可靠，尤其是内容包含特殊字符时。
- 调试 heredoc 截断时检查：文件末尾是否缺少内容，用 `wc -l` 对比预期行数。
- 这是跨平台踩坑：同样的 heredoc 在 bash 和 zsh 行为不同，macOS 默认 zsh 需要特别注意。
