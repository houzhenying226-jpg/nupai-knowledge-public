# KNOWLEDGE_CARD零搬运协议

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

两端 skill 配套：claude.ai 端负责“草稿生成”，OpenClaw 端负责“入库执行”。用户全程 1 次复制粘贴。支持三种触发模式：A 用户点名拆，B 列清单挑选，C 全部自动拆。多卡识别规则区分“应该拆”（不同主题/类型/根因）vs“不应拆”（同一坑的多次尝试 / 决策的前因后果 / 连贯操作流程）。

## 正文

> 记录日期：2026-04-30  风险等级：L3

## 背景
v1.1 方案 6.1 节的 nupai-knowledge-writer skill 设计是 OpenClaw 跟用户在飞书 5 轮问答后入库。但用户大量讨论实际发生在 claude.ai 网页端，OpenClaw 看不到 claude.ai 对话内容，5 轮问答相当于复述 5 次，违反“零搬运”原则。

## 内容
设计 KNOWLEDGE_CARD 协议作为 claude.ai 和 OpenClaw 之间的标准化交换格式。claude.ai 端 skill 从对话上下文提取信息，输出 KNOWLEDGE_CARD 标记块（含 frontmatter 5 字段 + 5 段正文），多卡用 === 分隔。OpenClaw 端 skill 检测到标记块，正则解析所有字段，跳过 5 轮问答，直接入库飞书表 A。claude.ai 端 skill 通过 claude.ai 的 Settings → Capabilities → Skills 上传 SKILL.md 启用，YAML frontmatter 必须包含 name 和 description。

## 结论
两端 skill 配套：claude.ai 端负责“草稿生成”，OpenClaw 端负责“入库执行”。用户全程 1 次复制粘贴。支持三种触发模式：A 用户点名拆，B 列清单挑选，C 全部自动拆。多卡识别规则区分“应该拆”（不同主题/类型/根因）vs“不应拆”（同一坑的多次尝试 / 决策的前因后果 / 连贯操作流程）。

## 影响
NuPai MemoryOps 增加跨端协议层。skill 文件位置：claude.ai 端走 Settings → Skills 上传，OpenClaw 端在 ~/.openclaw/workspace/skills/nupai-knowledge-writer/SKILL.md。两端通过 KNOWLEDGE_CARD 标记块解耦。

## 下次注意
跨 AI 平台的工作流设计，必须有标准化交换格式。光靠“AI 自己懂”不行，新会话/不同模型不知道格式。KNOWLEDGE_CARD 是 NuPai 系统设计语言，未来扩展（比如知识卡更新、知识卡删除）也走标记块协议。

