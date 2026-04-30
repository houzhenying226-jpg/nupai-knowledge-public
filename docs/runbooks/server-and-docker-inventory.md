# 服务器与 Docker 容器清单

> 类型：操作规程 | 项目：crm | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

NuPai 生产服务器（CRM/门店/Dify/Mac）与 Docker 容器完整清单，含端口、健康检查 URL、关键运维命令。

## 正文

# 服务器与 Docker 容器清单

## 服务器
| 名称 | IP | 用途 |
|------|-----|------|
| CRM 生产 | 39.106.83.79 | FastAPI+Next.js |
| 门店 | 39.96.216.33 | 门店后端 |
| Dify | 123.57.224.35 | AI 工作流 |

## CRM Docker 容器
| 容器 | 端口 | 作用 |
|------|------|------|
| nupai-fastapi | 8001 | 后端 API（注意不是 8000）|
| nupai-frontend | 3000 | Next.js |
| nupai-mysql | 3306 | 主数据库 |
| nupai-redis | 6379 | 缓存 |
| nupai-nginx | 80/443 | 反向代理 |

## 健康端点
- CRM API: http://39.106.83.79:8001/health
- Dify: http://123.57.224.35/health

## 注意
- 禁止 docker build --no-cache
- 生产 SSH 命令需查 OPS-COMMANDS.md 白名单
