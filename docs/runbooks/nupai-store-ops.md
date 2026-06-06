# 门店系统运维手册（nupai-store）

> 类型：运维手册 | 项目：nupai-store | 状态：生效
> 最后更新：2026-06-02 | 维护人：振英

---

## 📋 速查表

| 项目 | 值 |
|------|-----|
| **服务器** | root@39.96.216.33 |
| **项目路径** | /opt/nupai-store |
| **域名** | store.fesuntailor.net |
| **API 端口** | 8001（容器内 8000 → 宿主机 8001） |
| **Web 端口** | 8080 → nginx → 80 |
| **数据库** | PostgreSQL，用户 nupai，库名 nupai |
| **数据库容器** | nupai-store-postgres-1 |
| **部署方式** | Docker Compose + deploy.sh |

---

## 🔐 关键配置

以下配置写死在 `/opt/nupai-store/.env` 和 `docker-compose.yml` 中。修改后必须 `docker compose up -d`（不是 restart）才能生效。

### Redis

```
REDIS_PASSWORD=nupai2026redis
```

**⚠️ 致命坑**：`docker-compose.yml` 里 Redis command 必须带 `--requirepass nupai2026redis`，否则 API 连不上，全站 500。

正确配置（已写入 compose.yml 2026-06-02）：
```yaml
redis:
  image: redis:7-alpine
  command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru \
    --requirepass nupai2026redis \
    --save 900 1 --save 300 10 --save 60 10000 --appendonly no
```

- 已改用 RDB 快照，禁用 AOF（避免文件损坏导致循环重启）
- 禁止手动 `docker run redis`，一切走 compose

### 数据库

```
DB_HOST=postgres
DB_PORT=5432
DB_NAME=nupai
DB_USER=nupai
DB_PASSWORD=（见服务器 .env，同 Redis 密码）
```

### 登录账号

| 角色 | 用户名 | 密码 |
|------|--------|------|
| 平台管理员 | admin | admin123 |
| 销售员 | 13911222699 | nupai2026 |

---

## 🩺 常见故障排查

### 1. 全站 500 / API 不可用

```bash
# 先看 API 日志
docker logs nupai-store-api-1 --tail=50 2>&1 | grep -i error

# 如果看到 redis.exceptions.AuthenticationError → Redis 密码不匹配
# 检查 compose.yml 里 redis command 是否含 --requirepass
grep requirepass docker-compose.yml
# 如果没有 → 加上，重建
docker compose up -d redis && sleep 5 && docker restart nupai-store-api-1
```

### 2. Redis 反复重启 / 不健康

```bash
docker logs nupai-store-redis-1 --tail=20
# 常见原因：AOF 损坏 → 已改用 RDB，不再出现
# 如需清空 Redis 数据：
docker compose down redis && docker volume rm nupai-store_redis_data && docker compose up -d redis
```

### 3. SSL 证书过期

证书在宿主机 `/etc/letsencrypt/`，续期后用实体文件路径复制到 nginx：
```bash
# 不能用符号链接！docker cp 不支持
docker cp /etc/letsencrypt/archive/store.fesuntailor.net/fullchain2.pem \
  nupai-store-nginx-1:/etc/nginx/ssl/fullchain.pem
docker cp /etc/letsencrypt/archive/store.fesuntailor.net/privkey2.pem \
  nupai-store-nginx-1:/etc/nginx/ssl/privkey.pem
docker restart nupai-store-nginx-1
```

### 4. 磁盘满

```bash
# 安全检查
df -h /
du -sh /var/lib/docker/* | sort -rh | head -10

# 安全清理（不会删正在用的东西）
docker system prune -f
docker builder prune --all --force
```

---

## 🚀 部署流程

```bash
cd /opt/nupai-store
git pull origin main
./deploy.sh          # 自动构建 + 重启
# 验证
curl -s -o /dev/null -w '%{http_code}' http://localhost:8001/docs
```

---

## 📦 容器清单

| 容器 | 作用 | 健康检查 |
|------|------|----------|
| nupai-store-api-1 | FastAPI 后端 | `/docs` |
| nupai-store-web-1 | 前端静态文件 | 8080 |
| nupai-store-nginx-1 | 反向代理 + SSL | 80/443 |
| nupai-store-postgres-1 | 数据库 | pg_isready |
| nupai-store-redis-1 | 缓存/消息队列 | redis-cli ping |
| nupai-store-prometheus-1 | 监控采集 | — |
| nupai-store-grafana-1 | 监控面板 | — |
| nupai-store-alertmanager-1 | 告警 | — |
| nupai-store-dingtalk-bot-1 | 钉钉通知 | — |

---

## 📝 变更记录

| 日期 | 变更 | 原因 |
|------|------|------|
| 2026-06-02 | Redis 加 --requirepass | 手动重建丢密码导致全站 500 |
| 2026-06-02 | Redis 改用 RDB，禁用 AOF | AOF 201MB 文件损坏导致循环重启 |
| 2026-06-02 | compose.yml 固化 Redis 密码 | 禁止手动 docker run，防止配置漂移 |
