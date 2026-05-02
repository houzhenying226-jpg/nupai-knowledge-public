# ClaudeCodeMCP配置路径错误

> 类型：故障复盘 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

Claude Code CLI 的 MCP 配置位置是 ~/.claude.json（user scope）或项目根目录 .mcp.json（project scope）。推荐用 CLI 命令 claude mcp add memory -s user -- npx -y @modelcontextprotocol/server-memory，由 Claude Code 自适配版本写正确文件。验证

## 正文

> 记录日期：2026-04-29  风险等级：L2

## 背景
v1.0 方案 7.1 节让 Claude Code 把 MCP server 配置写在 ~/.claude/claude_desktop_config.json。三家模型交叉验证（GPT-5.5 / Opus / Gemini）一致指出此路径错误。

## 内容
~/.claude/claude_desktop_config.json 是 Claude Desktop 桌面应用的配置文件，Claude Code CLI 不读这个路径。如果按 v1.0 写法配置，npm 包装了但 Claude Code 运行时完全加载不到 MCP server，是典型"假成功"——表面上没报错，但功能完全不工作。

## 结论
Claude Code CLI 的 MCP 配置位置是 ~/.claude.json（user scope）或项目根目录 .mcp.json（project scope）。推荐用 CLI 命令 claude mcp add memory -s user -- npx -y @modelcontextprotocol/server-memory，由 Claude Code 自适配版本写正确文件。验证用 claude mcp list。

## 影响
v1.0 方案修订到 v1.1，第七章 MCP 配置部分整体重写。任何 NuPai 文档涉及 Claude Code MCP 的，都要检查路径是否正确。

## 下次注意
配置 Claude Code 任何东西时，先确认是 CLI 工具还是 Desktop 应用——两者配置文件完全不同。优先使用 CLI 命令而非手编 JSON，CLI 命令会自动找到正确路径。

