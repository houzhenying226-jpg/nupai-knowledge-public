# protect-files-hook加固方案

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

hook 加白名单外置 + 三层标准化 + symlink 防御。具体改动：(1) 新增 ~/.claude/hooks/claudemd-allowlist.txt，每行一个允许文件的绝对路径；(2) 检测路径前先 realpath 标准化（防 symlink 攻击：ln -s ~/.claude /tmp/trap 绕过保护）；(3) realpath 三层 fallback：realpath

## 正文

> 记录日期：2026-04-30  风险等级：L3

## 背景
v1.1 方案 4.4 节让 CC 写 nupai-crm / nupai-store 的项目级 CLAUDE.md，但 protect-files hook 拦截所有 CLAUDE.md（包括项目级），CC 写不进去。

## 内容
原 hook 用 PROTECTED_PATTERNS 数组 + grep -qiE "CLAUDE.md" 模式匹配，过保护——~/.claude/CLAUDE.md 应该保护（CC 自我注入意识形态入口），但项目级 nupai-crm/CLAUDE.md 是项目配置文件，本就该让 CC 维护。直接绕过 hook（用 ! 前缀）违反振英自己的安全设计。

## 结论
hook 加白名单外置 + 三层标准化 + symlink 防御。具体改动：(1) 新增 ~/.claude/hooks/claudemd-allowlist.txt，每行一个允许文件的绝对路径；(2) 检测路径前先 realpath 标准化（防 symlink 攻击：ln -s ~/.claude /tmp/trap 绕过保护）；(3) realpath 三层 fallback：realpath → python3 abspath → raw 字符串。豁免触发时 stderr 输出 ✅ 豁免日志便于排查。完整 diff 在 commit 303363c。

## 影响
全 NuPai 项目使用同一套 hook 体系。白名单配置外置后，未来加新项目（nupai-erp 等）只需编辑 .txt 文件，不用改 hook。

## 下次注意
设计安全 hook 时：(1) 必须区分"系统级保护文件"和"项目级配置文件"，不要一刀切；(2) 配置和代码分离，便于扩展；(3) 路径检查必须 realpath 防 symlink 绕过；(4) hook 改动是不可逆操作，必须先 osascript 通知振英 + 等确认 + commit infra/HOOK_CHANGES.md 备份。

