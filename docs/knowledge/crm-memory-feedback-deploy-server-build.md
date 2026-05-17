# 禁止在服务器构建 Next.js 镜像

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-17T20:15:36Z

## 摘要

deploy.sh 中 --build 导致服务器 OOM/超时宕机的根因与预防

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_deploy_server_build.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 根因（2026-05-11 真实事故）

`deploy.sh` 执行 `docker compose up -d --build --no-deps fastapi nextjs cortex`，每次部署都在生产服务器上实时构建 Next.js。
Next.js 完整构建需要 2-3GB 内存 + 10 分钟以上。
GitHub Actions `command_timeout: 10m`（默认）在构建中途切断 SSH 连接，Docker 容器停而未启，nginx 无法连接后端 → 504 宕机。

## 已修复（PR #546）

- `deploy.sh`：从 `--build` 列表中移除 `nextjs`（fastapi/cortex 保留，因为是 Python 层，构建快）
- `deploy.yml`：`command_timeout: 300s` → `command_timeout: 30m`

## 铁律

- **禁止**在部署脚本中对 nextjs 使用 `--build`
- 前端有重大变更需要重建镜像时，手动 SSH 到服务器在低峰期执行：
  ```bash
  cd /opt/nupai-crm && docker compose up -d --build --no-deps nextjs
  ```
- **禁止**连续快速合并多个含前端变更的 PR（两次构建叠加必宕机）
- 每次 PR 合并后等待 `curl /health` 返回新版本号再合并下一个

## 触发条件识别

以下情况会导致 Docker 缓存失效并触发完整重建（高风险）：
- 修改了 `frontend/` 下任何 `.tsx/.ts` 文件
- 修改了 `frontend/package.json`
- 修改了 `frontend/Dockerfile`

## 长期治本方向

在 GitHub Actions Runner（7GB RAM）中构建 nextjs 镜像并推送到 ghcr.io，
服务器只执行 `docker compose pull && docker compose up -d`，彻底不在服务器构建。

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

