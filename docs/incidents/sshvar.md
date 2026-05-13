# SSH双引号命令中$VAR服务器端为空

> 类型：故障复盘 | 项目：nupai-store | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

用 heredoc 通过 stdin 管道传配置，不使用变量：
```PLAIN_TEXT
cat << 'NGINX_CONF' | ssh root@39.96.216.33 'NGINX=$(docker ps --format "{{.Names}}" | grep -i nginx | head -1) && docker exec -i $NGINX sh -c "cat > /etc

## 正文

> 记录日期：2026-05-13  风险等级：L3

## 背景
为避免 nginx 配置的 shell 转义问题，将配置 base64 编码后通过 SSH 传到服务器再解码写入。结果服务器收到空字符串，nginx conf 被清空，整个站点回退到旧配置。

## 内容
- 错误写法（fix-api-tunnel.sh）：
```PLAIN_TEXT
CONF_B64=$(echo "$CONF" | base64)   # 本地Mac上执行，得到base64字符串
ssh root@39.96.216.33 "  echo \$CONF_B64 | base64 -d | docker exec -i $NGINX sh -c 'cat > /etc/nginx/conf.d/xxx.conf'"
```
- 根因：\$CONF_B64 中 \$ 转义后在服务器端变成 $CONF_B64，服务器上该变量未定义 → 值为空字符串
- echo "" \| base64 -d 输出空，cat \> 写入空文件
- nginx -t 对空文件不报错（空 conf 文件合法），reload 成功但配置失效
- 症状：nginx 报 "nginx OK" 但请求返回 404，因为 vhost 配置消失了

## 结论
用 heredoc 通过 stdin 管道传配置，不使用变量：
```PLAIN_TEXT
cat << 'NGINX_CONF' | ssh root@39.96.216.33 'NGINX=$(docker ps --format "{{.Names}}" | grep -i nginx | head -1) && docker exec -i $NGINX sh -c "cat > /etc/nginx/conf.d/dev-sidebar.conf" && docker exec $NGINX nginx -t && docker exec $NGINX nginx -s reload'server {    listen 80;    server_name store.ifeishen.com;    ...}NGINX_CONF
```

## 影响
- fix-api-tunnel.sh（已废弃，逻辑并入 deploy-sidebar.sh）
- 任何需要把本地变量通过 SSH 传到服务器的场景

## 下次注意
本地变量要传到 SSH 远端，只有两种安全方式：① heredoc 管道（cat <<'EOF' \| ssh ...）② 把值直接拼进 SSH 命令字符串（不转义 $，让本地展开）。\$VAR 的写法一定是在服务器端读变量，本地变量无法传递。

