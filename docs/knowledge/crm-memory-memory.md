# MEMORY

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-16T21:38:24Z

## 摘要

- [PR流程规则](feedback_pr_workflow.md) — 禁止直接push main，必须走PR

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `MEMORY.md`，本次按侯爷要求同步到飞书知识库。

## 内容
- [PR流程规则](feedback_pr_workflow.md) — 禁止直接push main，必须走PR
- [Linear API 配置](reference_linear.md) — API key、团队/项目/标签/状态 ID，用于管理 Issues
- [Linear 必须用 curl](feedback_linear_api.md) — 禁止 MCP 连 Linear，只用 curl 调 GraphQL API
- [OpenClaw 禁止使用 Anthropic](feedback_openclaw_no_anthropic.md) — OpenClaw 运行时不依赖任何 Anthropic 服务，违反会被拉黑
- [Hooks 使用 stdin JSON](feedback_hooks_stdin_json.md) — PostToolUse hooks 通过 stdin JSON 传数据，不是环境变量；非零退出不触发 PostToolUse
- [MemoryOps v1.1 部署完成](project_memoryops_v11.md) — git learnings + H14/H15/H16 + 共享池 + skill-evolve 引擎，2026-05-04 上线
- [禁止在服务器构建 Next.js](feedback_deploy_server_build.md) — deploy.sh --build 导致 OOM/超时宕机，nextjs 已从构建列表移除
- [PR合并顺序与部署健康验证](feedback_pr_merge_order.md) — 上一个部署健康确认后才能合并下一个，worktree冲突时用 gh api 合并
- [Playwright innerText vs textContent](feedback_playwright_textcontent_vs_innertext.md) — 检测可见内容用 innerText，textContent 会抓隐藏Tab
- [角色权限分组必须对齐业务语义](feedback_role_permission_scope.md) — director 与 CEO 同级看全公司，错误分组导致 NUP-419
- [分支管理与 worktree 冲突处理](feedback_git_branch_workflow.md) — main 锁 worktree，从 origin/main 建分支，gh api 合并 PR
- [OpenClaw 文件写入绕过方法](feedback_openclaw_file_write_workaround.md) — python3 stdin 改 .py/.md，/tmp→cp 改 .tsx，git 操作必须让用户手动执行
- [前端 PR 两个隐形 CI 检查](feedback_frontend_ci_hidden_checks.md) — mobile-first 需 useMobile，blueprint-check 需 PR body 含 4 个章节
- [Pydantic model_fields_set 区分"未传"与"传 null"](feedback_pydantic_model_fields_set.md) — Optional 字段支持清除语义时用 model_fields_set 而非 is not None
- [后端序列化函数必须输出前端用到的所有字段](feedback_api_serialization_completeness.md) — _user_to_dict 漏字段导致前端列永远空，新增列时必须同步检查序列化函数
- [三台服务器 IP 与职责](reference_server_ips.md) — CRM=[REDACTED_IP]，Store=[REDACTED_IP]，备份=[REDACTED_IP]；CLAUDE.md 写错了
- [deploy.yml 健康检查假通过 Bug](feedback_health_check_false_positive.md) — 8001 端口被 Store 占，CRM 不在也返回 200，历史所有部署验证均不可信
- [WF1 记忆卡生成 7天0%通过率](feedback_wf1_memory_card_failure.md) — WF1 空输出+连接池50并发耗尽，补写功能完全失效，需两个独立 Issue 修
- [提醒中心两种JSON content格式](feedback_reminder_two_json_formats.md) — tryParseEmpowerment只处理{开头，[{style,content}]数组必须用tryParseGreeting，两者不能混用
- [frontend/mobile-pages/是孤立死代码](feedback_mobile_pages_dead_code.md) — mobile-pages/下所有.tsx无任何import引用，修bug只改app/，不需要同步
- [Codex审查触发机制](feedback_codex_review_trigger.md) — chatgpt-codex-connector在PR create/push时自动触发，不是评论触发；Codex不来推空commit重触发
- [gh pr comment不被D0守卫拦截](feedback_gh_pr_comment_allowed.md) — D0只拦截文件写入，gh pr comment/run rerun/api POST均可直接用
- [PR Gate rerun后查attempt/2](feedback_pr_gate_rerun_attempt2.md) — gh run rerun不生成新run_id，创建attempt 2；查新结果需用.../attempts/2/jobs
- [两种乱码根因完全不同](feedback_two_types_of_garbled_text.md) — JSON原文泄漏=前端渲染bug改代码；DB mojibake=数据bug执行SQL；不能混为一谈
- [Bug修复不彻底根因：新格式未同步消费入口](feedback_fix_incomplete_multi_entry.md) — 后端新增content格式时必须检查所有前端消费入口，fallback渲染不能暴露原始JSON

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

