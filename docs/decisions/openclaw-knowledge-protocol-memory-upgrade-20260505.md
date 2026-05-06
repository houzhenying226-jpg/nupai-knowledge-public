# OpenClaw 知识卡协议入口与本地记忆层升级

> 类型：架构决策 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:30:00Z

## 摘要

知识卡入库必须作为确定性协议处理，直接入库并双写本地事件日志和 Markdown 镜像；自然语言任务继续交给 registerTool。

## 正文

> 记录日期：2026-05-05  风险等级：L2

## 背景
用户在飞书贴知识卡时，旧链路会把内容继续交给 LLM，出现静默或错误回复。

## 内容
nupai-oc-router 现在在 before_dispatch 里处理 KNOWLEDGE_CARD 并返回 handled:true，同时注册 inbound_claim。knowledge_writer.py 写飞书表 A 后，生成 ~/.openclaw/knowledge/events.jsonl、index.jsonl 和 cards/<project>/<slug>.md。

## 结论
协议型消息不走 LLM，自然语言任务走 registerTool，长期记忆不只依赖飞书表 A。

## 影响
以后知识卡应收到明确入库回执，并能用 rg 检索本地镜像。

## 下次注意
不能再用关键词补丁处理协议；协议入口要有 handled:true、事件日志、镜像文件和验证命令。

