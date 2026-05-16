# WF1 记忆卡生成 7天0%通过率 + 连接池耗尽

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-16T10:04:44Z

## 摘要

调度器补写记忆卡实际失败，两个独立根因

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_wf1_memory_card_failure.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 现象（2026-05-12 发现）

```
Dify WF WF1 returned empty outputs (attempt 1)
output_validator 7日通过率 0.0% < 80% (0/48)

QueuePool limit of size 30 overflow 10 reached,
connection timed out, timeout 30.00
```

## 根因 1：WF1 返回空输出（Dify 侧问题）

- Dify `http://[REDACTED_IP]/v1/workflows/run` 返回 200 但 outputs 为空
- output_validator 7天内 48 次调用全部失败（0/48）
- 需要检查 Dify 后台 WF1 工作流配置是否损坏

## 根因 2：DB 连接池耗尽（代码侧问题）

`job_retry_pending_visits`（NUP-418）每批并发启动 50 个 WF1 asyncio.create_task()：
- 50 个任务同时等待 Dify HTTP 响应（慢/超时）
- 每个任务完成后做 DB 写入，占用 DB 连接
- SQLAlchemy QueuePool size=30 + overflow=10 = 最多 40 连接，被 50 并发打满

## 影响

调度器每 30 分钟运行但实际不生成任何记忆卡。
补写功能（NUP-418 设计的核心价值）完全失效。

## 修复方向（两个独立 Issue）

**Issue A（P0）**：排查 WF1 Dify 工作流为何返回空 outputs
- 登录 Dify 后台 → WF1 → 查看最近执行日志
- 可能原因：工作流变量名变更、输出字段改了、Dify 版本升级

**Issue B（P1）**：降低调度器并发数
```python
# 当前（危险）
BATCH_SIZE = 50
# 改为
BATCH_SIZE = 10  # 或加信号量限流
```
或加 asyncio.Semaphore(10) 限制最大并发 WF1 调用数。

## 相关文件

- `backend/app/services/scheduler.py` `job_retry_pending_visits()`
- `backend/app/api/v1/visits.py` `_trigger_wf1_analysis()`

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

