# Store环境变量Key清单

> 类型：操作规程 | 项目：nupai-store | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

.env 修改后必须 recreate 不是 restart 才会重载（基础设施清单 §17 铁律）。deploy.sh 部署前自动备份到 /tmp/.env.deploy.bak，每日异地同步到职业装1（123.57.224.35）。敏感 Key 不得写进 docker-compose.yml 或 git。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
Store 业务容器读取 /opt/nupai-store/.env，里面包含数据库、AI 服务、阿里云、钉钉等敏感配置。排查“为什么集成失败”先看 Key 是否齐全。

## 内容
数据库/Redis（敏感）：DATABASE_URL、DB_HOST/DB_PORT/DB_NAME/DB_USER/DB_PASSWORD、REDIS_URL、REDIS_PASSWORD。 应用密钥（敏感）：SECRET_KEY、AES_KEY、JWT_EXPIRE_MINUTES（非敏感）。 AI 服务（敏感）：DEEPSEEK_API_KEY、DIFY_API_KEY、DIFY_BASE_URL、DIFY_CHAT_API_KEY、BAILIAN_API_KEY、BAILIAN_BASE_URL。 钉钉（敏感）：DINGTALK_APP_KEY、DINGTALK_APP_SECRET、DINGTALK_CLIENT_ID、DINGTALK_CLIENT_SECRET、DINGTALK_OPERATOR_ID。 阿里云短信（敏感）：ALIYUN_SMS_ACCESS_KEY_ID、ALIYUN_SMS_ACCESS_KEY_SECRET、ALIYUN_SMS_SIGN、ALIYUN_SMS_TEMPLATE_CODE。 阿里云 OSS（敏感）：OSS_ACCESS_KEY_ID、OSS_ACCESS_KEY_SECRET、OSS_BUCKET、OSS_ENDPOINT、OSS_URL_PREFIX。 监控（非敏感）：SENTRY_DSN、VITE_SENTRY_DSN。 业务参数（非敏感）：DEFAULT_TENANT_ID、WEWORKMSG_BASE_URL（默认 http://localhost:8888）、WECOM_SYNC_CRON_HOURS（默认 "9,13,18"）、WECOM_VOICE_TRANSCRIBE_ENABLED（默认 true）。

## 结论
.env 修改后必须 recreate 不是 restart 才会重载（基础设施清单 §17 铁律）。deploy.sh 部署前自动备份到 /tmp/.env.deploy.bak，每日异地同步到职业装1（123.57.224.35）。敏感 Key 不得写进 docker-compose.yml 或 git。

## 影响
Store 所有外部集成（Dify/百炼/钉钉/OSS/短信/Sentry/企微）；deploy.sh 流程；环境变量泄露恢复路径；新增集成时 Key 注入位置。

## 下次注意
新增 Key 一律走 .env 不要硬编码；改 .env 后用 recreate；Key 命名按 {服务}_{用途} 规范（基础设施清单 §16）；Key 泄露立即在对应平台轮换 + 异地备份同步。

