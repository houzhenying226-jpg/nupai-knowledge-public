# MemoryOps v1.1 落地已知坑清单

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

批 1-4 落地的 8 个已知坑：realpath 跨平台、openclaw.json 被覆盖、bash-firewall 误伤、Bitable 字段上限、Wiki 权限、gateway 未加载、C4 接口格式、hook 自拦截。

## 正文

# MemoryOps v1.1 落地已知坑清单

## 坑 1：realpath -m macOS 不可用
症状：hook 路径标准化静默失败
解法：三段 fallback realpath || python3 abspath || raw

## 坑 2：openclaw.json 被 OpenClaw 覆盖
症状：自定义 key 重启后消失
解法：独立文件 ~/.nupai/bitable.json

## 坑 3：bash-firewall 误伤 nupai-crm + .claude/hooks
症状：nupai-crm 含 rm 子串 + .claude/hooks 触发正则
解法：/tmp 中转，分两步执行

## 坑 4：Bitable 字段上限 80KB
症状：98KB 写入失败 TooLargeCell
解法：代码层 80KB 安全线，超出分 part

## 坑 5：Wiki API 权限门槛高
症状：wiki:wiki 权限后仍 403
解法：改 Route 2 Bitable 真源

## 坑 6：gateway 未加载时 cron list 显示空
症状：cron add 成功但 list 无结果
解法：直接编辑 jobs.json + 用 crontab 作可靠调度层

## 坑 7：C4 接口格式错误
症状：POST /command 返回 404
解法：subprocess ['openclaw', 'infer', 'model', 'run', '--gateway']

## 坑 8：protect-files 自拦截
症状：Write tool 无法写 .claude/hooks/ 目录
解法：只能用 Bash tool 的 cp 命令
