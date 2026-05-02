# deploy.sh无条件rebuild副作用

> 类型：故障复盘 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

当前接受这个设计（改造成本未排期），但运维侧要清楚副作用：① 文档 PR 合并也要看 deploy.sh 是否会跑挂 ② 镜像加速器/阿里云源/HEALTHCHECK 三道铁律必须配齐才能让“无条件 rebuild” 安全 ③ 任何 deploy 失败先看是不是网络问题不是代码问题。

## 正文

> 记录日期：2026-04-20  风险等级：L3

## 背景
基础设施清单 §18.4 复盘 deploy.sh 设计：只比较 git commit 哈希（PREV_COMMIT != NEW_COMMIT 就触发 docker-compose up -d --build），不区分代码改动还是文档改动。这意味着 PR#67/#68 这种纯文档 PR 也会重建镜像。

## 内容
deploy.sh L95-107 的逻辑：拿当前 main 的 commit 和上次部署记录的 commit 比较；不同 → 触发 docker compose -f ... -f ... up -d --build；相同 → 跳过。副作用：每次 deploy 都可能因 Docker Hub / apt 网络问题失败（即使代码零变更）；纯文档 PR 也应视作“需部署”来评估风险；STO-203 镜像加速器问题在这种场景下放大（每个 deploy 都要 build）。未来改进方向：可按 git diff --name-only 判断，仅在 backend/ / requirements.txt / Dockerfile* 改动时 rebuild。

## 结论
当前接受这个设计（改造成本未排期），但运维侧要清楚副作用：① 文档 PR 合并也要看 deploy.sh 是否会跑挂 ② 镜像加速器/阿里云源/HEALTHCHECK 三道铁律必须配齐才能让“无条件 rebuild” 安全 ③ 任何 deploy 失败先看是不是网络问题不是代码问题。

## 影响
所有 PR 合并后的部署评估、deploy.sh 改造排期、文档 PR 风险评估、镜像加速器/Dockerfile 源配置的优先级（变成硬前提）、CI/CD 流程。

## 下次注意
文档 PR 合并前也评估“deploy.sh 跑挂会怎样”；deploy 失败先排查 Docker Hub 网络/apt 网络（不是先怀疑代码）；未来改造 deploy.sh 时按 git diff --name-only 智能判断；镜像加速器 + 阿里云源 + 非 curl HEALTHCHECK 三件套必须先齐全。

