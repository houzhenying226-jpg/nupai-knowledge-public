# Docker容器内[127.0.0.1](127.0.0.1)≠宿主机loopback

> 类型：故障复盘 | 项目：nupai-store | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

修改 nginx proxy_pass 目标为 Docker gateway IP \+ iptables DNAT 重定向：
```PLAIN_TEXT
# nginx conf.d/dev-sidebar.conflocation /api/ {    proxy_pass http://10.255.13.1:8081/api/;}# 宿主机上执行（打通路由）echo 1 > /proc/s

## 正文

> 记录日期：2026-05-13  风险等级：L4

## 背景
nginx 跑在 Docker 容器内，配置 proxy_pass 到 [127.0.0.1:8081](127.0.0.1:8081)，期望转发到宿主机上的 SSH 隧道。结果一直 502 Bad Gateway。

## 内容
- 错误症状：nginx proxy_pass [http://127.0.0.1:8081/api/](http://127.0.0.1:8081/api/) → 502 Bad Gateway
- 根因：容器内的 [127.0.0.1](127.0.0.1) 是容器自己的 loopback，不是宿主机的 [127.0.0.1](127.0.0.1)
- 宿主机对容器暴露的 IP 是 Docker bridge gateway，nupai-store 服务器上实测为 [10.255.13.1](10.255.13.1)
- 这个 IP 因 Docker 网络配置不同而异（常见值：[172.17.0.1](172.17.0.1) / [172.20.0.1](172.20.0.1) / [10.255.13.1](10.255.13.1)）
- SSH 隧道绑定在宿主机 [127.0.0.1:8081](127.0.0.1:8081)，容器默认无法直接访问

## 结论
修改 nginx proxy_pass 目标为 Docker gateway IP \+ iptables DNAT 重定向：
```PLAIN_TEXT
# nginx conf.d/dev-sidebar.conflocation /api/ {    proxy_pass http://10.255.13.1:8081/api/;}# 宿主机上执行（打通路由）echo 1 > /proc/sys/net/ipv4/conf/all/route_localnetiptables -t nat -D PREROUTING -p tcp --dport 8081 -j DNAT --to-destination 127.0.0.1:8081 2>/dev/null || trueiptables -t nat -A PREROUTING -p tcp --dport 8081 -j DNAT --to-destination 127.0.0.1:8081
```

## 影响
- deploy-sidebar.sh 中 nginx 配置段
- fix-api-tunnel.sh
- 任何在 Docker 容器内向宿主机服务发起代理请求的 nginx 配置

## 下次注意
nginx 在 Docker 容器内时，proxy_pass 绝对不能写 [127.0.0.1](127.0.0.1)，必须用 Docker gateway IP（先 ip route \| grep default 确认）\+ iptables DNAT \+ route_localnet 三件套。

