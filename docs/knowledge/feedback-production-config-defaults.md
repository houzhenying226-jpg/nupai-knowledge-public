# L18 · 生产环境配置三个关键默认值：workers/Sentry采样/启动延迟

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

生产环境三个关键默认值必须配置：workers=4、Sentry 5% 采样、任务延迟 90s、Next.js memory 1g。

## 正文

> 记录日期：2026-05-22  风险等级：L2

## 背景
uvicorn workers=1 串行处理；Sentry 100% 采样每请求 +20ms 开销；定时任务无启动延迟压垮启动。

## 内容
Dockerfile: --workers 4。main.py: traces_sample_rate=0.05, profiles_sample_rate=0.05。scheduler: next_run_time=datetime.now() + timedelta(seconds=90)。docker-compose: Next.js memory: 1g。

## 结论
生产环境三个关键默认值必须配置：workers=4、Sentry 5% 采样、任务延迟 90s、Next.js memory 1g。

## 影响
吞吐量 4x 提升；Sentry 性能开销降低 95%；启动稳定性提升避免任务并发冲突。

## 下次注意
部署前检查 workers=4、Sentry 5%、任务延迟 90s、Next.js memory 1g 四个配置项。

