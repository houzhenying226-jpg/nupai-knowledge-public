# 知识库同步链推送中断排查与修复（2026-09-24）

## 结论
知识库两路自动推送（30 分钟同步 `feishu-wiki-sync` + 每日 22:00 快照 `status-snapshot`）长期「本地提交成功、推送不落库」。根因 = 推送通道依赖图形会话钥匙串凭据，定时环境取不到 → 推送 3 连败；且失败被静默。已把两个仓库的推送通道切换为 SSH 免密（deploy_key），13:00 轮次自然自证通过。

## 症状（修复前）
- 7/1 起 origin/main 不再收到脚本推送（期间仅人工/助理手动补推）。
- `feishu-wiki-sync`：定时环境每轮在提交前崩溃（gitleaks 裸名调用 + 残留文件拦截），数月近零产出，积压 93 个文档。
- `status-snapshot`：推送失败仅发一条飞书告警，随后仍记「正常」——静默失败，运行历史全绿掩盖了约 3 个月的推送中断。

## 根因（两层）
1. 脚本级：定时环境 PATH 不含 `/opt/homebrew/bin`，裸名 `gitleaks` 调用抛 FileNotFoundError → 崩在提交前。
2. 环境级：推送走 https，凭据由 `gh auth git-credential`（读系统钥匙串）提供；定时任务在非图形会话中取不到钥匙串 → `git push` 立即失败 ×3。手动运行则一切正常——形成「人跑就行、定时跑不行」。

## 修复
- （上午）`feishu_wiki_sync.py`：gitleaks 改绝对路径 `/opt/homebrew/bin/gitleaks`；隔离残留拦截文件；补推积压 93 个提交。
- （下午）推送通道切换：
  - `~/.ssh/config` 增加 `Host github.com → IdentityFile ~/.ssh/deploy_key`（免密；备份见 `~/.ssh/config.bak-<epoch>`）；
  - `nupai-knowledge`、`nupai-knowledge-public` 两仓 remote 由 https 改为 `git@github.com:…`。
- 失败可见性：
  - 两脚本推送失败信息带上 stderr 末尾；
  - `feishu-wiki-sync` 失败时打印到运行日志（`/tmp/nupai-c1.log`）；
  - `status-snapshot` 推送失败不再记「正常」，改记「异常」。

## 验证
- 最小环境实测：`env -i HOME=… PATH=/usr/bin:/bin git push`（临时分支）→ rc=0；临时分支已删除。
- 自然自证：13:00 轮次自带推送成功——origin `82d38a2`，reflog `13:00:09 update by push`，运行历史 ok=True，日志 `done: 68 file(s)`。
- 第二道自证：今晚 22:00 `status-snapshot` 轮次（预期 origin 出现 `snapshot(status): daily update 2026-09-24`）。

## 影响文件
- `~/.openclaw/scripts/feishu_wiki_sync.py`、`~/.openclaw/scripts/status_snapshot.py`
- `~/.ssh/config`（新增段）、两仓 `.git/config`（remote 改 SSH）

## 备注
- 其它仓库若有「定时任务 + 钥匙串凭据」同样模式，存在同类隐患（本次未逐一审计）。
- 推送失败时的 stderr 已保留在告警与日志中，后续失败可凭原文定位。
