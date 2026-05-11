# 禁止在服务器构建 Next.js 镜像

> 类型：操作规程 | 项目：nupai-crm | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-11T10:30:00Z

## 摘要

deploy.sh 禁止对 nextjs 使用 --build，避免生产服务器构建 Next.js 导致超时和 504 宕机。

## 正文

> 记录日期：2026-05-11  风险等级：L4

## 背景
2026-05-11 真实事故：deploy.sh 执行 docker compose up -d --build --no-deps fastapi nextjs cortex，每次部署都在生产服务器实时构建 Next.js。Next.js 完整构建需要 2-3GB 内存和 10 分钟以上，GitHub Actions command_timeout 默认 10m 在构建中途切断 SSH，容器停而未启，nginx 无法连接后端导致 504 宕机。

## 内容
已在 PR #546 修复：deploy.sh 从 --build 列表移除 nextjs（fastapi/cortex 保留）；deploy.yml command_timeout 从 300s 调整到 30m。触发高风险的条件包括修改 frontend/ 下 .tsx/.ts、frontend/package.json、frontend/Dockerfile。前端重大变更需低峰期手动重建 nextjs 镜像，生产服务器地址统一脱敏为 <CRM_SERVER_IP>。

## 结论
禁止在部署脚本中对 nextjs 使用 --build。PR 合并后必须等待 /health 返回新版本号再合并下一个。长期治本方向是在 GitHub Actions Runner 构建 nextjs 镜像并推送到 ghcr.io，服务器只 pull/up。

## 影响
避免部署时生产服务器内存不足、SSH 超时、容器停而未启、nginx 504。降低连续合并前端 PR 的宕机风险。

## 下次注意
禁止连续快速合并多个含前端变更的 PR；每次合并后等待健康检查返回新版本号；涉及前端构建时优先走 CI 构建镜像方案。

