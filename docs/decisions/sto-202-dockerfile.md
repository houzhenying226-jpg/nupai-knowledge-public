# STO-202 Dockerfile阿里云源

> 类型：架构决策 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

backend/Dockerfile.prod 必须补阿里云源；dingtalk-bot/Dockerfile pip 加阿里云源；新建 Dockerfile 一律按以上模板起手。这和 STO-203 镜像加速器叠加才能保证 build 在大陆稳定。

## 正文

> 记录日期：2026-04-20  风险等级：L3

## 背景
STO-202：自建 Dockerfile build 时 apt-get 和 pip install 默认走国外源，在中国大陆服务器经常超时，即使 Docker Hub 镜像层已通过加速器拉到。需要把 apt 和 pip 都换阿里云源。

## 内容
所有自建 Dockerfile 必须同时配阿里云 apt + pypi 源：FROM python:3.12-slim；apt 换阿里云：sed -i "s|deb.debian.org|mirrors.aliyun.com|g" /etc/apt/sources.list.d/debian.sources；apt 装包：apt-get update && apt-get install -y --no-install-recommends gcc libpq-dev ...；pip 换阿里云：pip install --no-cache-dir -i https://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com -r requirements.txt。当前状态（2026-04-20，基础设施清单 §18.2）：backend/Dockerfile（dev 版）：✅ 已有阿里云源；backend/Dockerfile.prod（prod 版）：❌ 缺阿里云源（STO-202 跟进项）；dingtalk-bot/Dockerfile：❌ 无 apt，但 pip 也无阿里云源。

## 结论
backend/Dockerfile.prod 必须补阿里云源；dingtalk-bot/Dockerfile pip 加阿里云源；新建 Dockerfile 一律按以上模板起手。这和 STO-203 镜像加速器叠加才能保证 build 在大陆稳定。

## 影响
所有自建 Dockerfile（backend/Dockerfile、backend/Dockerfile.prod、dingtalk-bot/Dockerfile、CRM nextjs Dockerfile、CRM cortex/Dockerfile 等）；docker compose build 稳定性；deploy.sh 成功率；新建容器服务的 Dockerfile 模板。

## 下次注意
新建 Dockerfile 起手先抄此模板（apt + pypi 双换源）；CR Dockerfile 改动检查源是否换；backend/Dockerfile.prod 立刻补；dingtalk-bot/Dockerfile pip 也补；以后所有 NuPai 自建镜像 Dockerfile 都强制走阿里云源。

