# NuPai 9 层防护体系

> 类型：架构决策 | 项目：crm | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

9层防线：代码/测试/PR审查/CI/部署/运行时/Hook/凭据/知识层。每层有明确负责方和失效兜底机制。

## 正文

# NuPai 9 层防护体系

| 层 | 名称 | 机制 |
|---|------|------|
| L1 | 代码层 | ruff+mypy+bandit |
| L2 | 测试层 | pytest+Playwright |
| L3 | PR 审查 | reviewer+codex adversarial |
| L4 | CI/CD | gitleaks+构建验证 |
| L5 | 部署 | deploy.yml+健康检查 |
| L6 | 运行时 | Sentry+30min 告警窗 |
| L7 | Hook | protect-files+bash-firewall |
| L8 | 凭据 | gitleaks pre-push+.gitignore |
| L9 | 知识 | MemoryOps v1.1 起手式 |

## L7 Hook 详解
- protect-files.sh：阻止修改关键文件（豁免白名单 claudemd-allowlist.txt）
- bash-firewall.sh：拦截危险 shell 命令

## L8 凭据防护
- .gitignore 排除所有 .env*/secret/password 文件
- gitleaks pre-push hook + GitHub Action 双层

## 失效兜底
- L7 绕过 → L4 CI 兜底
- 所有层失效 → 凭据轮换 + git filter-repo 历史清除
