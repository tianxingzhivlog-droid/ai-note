# OpenClaw Docker 快速部署指南

**版本**: 2026.4.2  
**创建时间**: 2026-04-06  
**标签**: docker, deployment, quickstart, configuration

---

## 📋 概述

本文档记录使用 Docker 快速部署 OpenClaw 容器的完整流程，包括关键配置步骤和常见问题解决方案。

---

## 🚀 快速启动

### 方案一：Docker Run（推荐）

```bash
# 一键启动命令
docker run -d \
  --name stock \
  -p 9999:18789 \
  -v ~/.openclaw-stock:/root/.openclaw \
  -e OPENCLAW_GATEWAY_PORT=18789 \
  ghcr.io/openclaw/openclaw:2026.4.2
```

### 方案二：Docker Compose

```yaml
version: '3.8'
services:
  openclaw:
    image: ghcr.io/openclaw/openclaw:2026.4.2
    container_name: stock
    ports:
      - "9999:18789"
    volumes:
      - ~/.openclaw-stock:/root/.openclaw
    environment:
      - OPENCLAW_GATEWAY_PORT=18789
    restart: unless-stopped
```

---

## ⚠️ 关键配置（必须执行）

### 步骤 1: 配置绑定模式

```bash
docker exec stock node dist/index.js config set gateway.bind "lan"
```

**原因**: 默认绑定到 `127.0.0.1`（容器内部），设置为 `lan` 后绑定到 `0.0.0.0`，允许宿主机访问。

### 步骤 2: 配置允许的源地址

```bash
docker exec stock node dist/index.js config set gateway.controlUi.allowedOrigins '["http://localhost:9999","http://127.0.0.1:9999"]'
```

**原因**: 防止浏览器访问时报 "origin not allowed" 错误。

### 步骤 3: 设置认证 Token

```bash
docker exec stock node dist/index.js config set gateway.auth.token "hope"
```

### 步骤 4: 重启容器

```bash
docker restart stock
```

---

## ✅ 验证步骤

### 检查容器状态

```bash
docker ps | grep stock
```

### 检查端口映射

```bash
docker port stock
```

### 检查日志（确认监听 0.0.0.0）

```bash
docker logs stock --tail 20
```

**正确输出示例**:
```
listening on ws://0.0.0.0:18789
Gateway is binding to a non-loopback address
```

### 测试连接

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:9999/
# 应返回 200
```

---

## 🔑 访问信息

| 项目 | 值 |
|------|-----|
| 访问地址 | http://localhost:9999/ |
| Token | hope |
| 容器名称 | stock |
| 数据卷 | ~/.openclaw-stock |
| 内部端口 | 18789 |
| 外部端口 | 9999 |

---

## 🐛 常见问题

### 1. "origin not allowed" 错误

**症状**: 浏览器控制台显示 `origin not allowed (open the Control UI from the gateway host or allow it in gateway.controlUi.allowedOrigins)`

**解决**:
```bash
docker exec stock node dist/index.js config set gateway.controlUi.allowedOrigins '["http://localhost:9999","http://127.0.0.1:9999"]'
docker restart stock
```

### 2. "pairing required" 错误

**症状**: 登录后仍提示需要配对

**解决**:
```bash
# 查看待处理设备
docker exec stock node dist/index.js devices list

# 批准设备（替换 <request-id> 为实际 ID）
docker exec stock node dist/index.js devices approve <request-id>
```

### 3. 无法访问 9999 端口

**检查步骤**:
```bash
# 检查端口是否被占用
lsof -i :9999

# 检查容器状态
docker ps | grep stock

# 检查网关绑定
docker exec stock node dist/index.js config get gateway.bind
# 应返回 "lan"
```

### 4. 容器启动后自动退出

**检查日志**:
```bash
docker logs stock --tail 50
```

**常见原因**:
- 配置文件格式错误
- 端口冲突
- 权限问题

---

## 📝 完整初始化脚本

复制以下脚本一键完成所有配置：

```bash
#!/bin/bash

# 1. 创建并启动容器
docker run -d \
  --name stock \
  -p 9999:18789 \
  -v ~/.openclaw-stock:/root/.openclaw \
  -e OPENCLAW_GATEWAY_PORT=18789 \
  ghcr.io/openclaw/openclaw:2026.4.2

echo "⏳ 等待容器初始化..."
sleep 5

# 2. 配置绑定模式
docker exec stock node dist/index.js config set gateway.bind "lan"

# 3. 配置允许的源
docker exec stock node dist/index.js config set gateway.controlUi.allowedOrigins '["http://localhost:9999","http://127.0.0.1:9999"]'

# 4. 设置 Token
docker exec stock node dist/index.js config set gateway.auth.token "hope"

# 5. 重启容器
docker restart stock

echo "⏳ 等待容器重启..."
sleep 5

# 6. 验证
echo "📊 容器状态:"
docker ps | grep stock

echo ""
echo "🔗 访问地址：http://localhost:9999/"
echo "🔑 Token: hope"
```

---

## 🔗 相关资源

- **官方文档**: https://docs.openclaw.ai/install/docker
- **GitHub 镜像**: https://github.com/openclaw/openclaw/pkgs/container/openclaw
- **社区**: https://discord.com/invite/clawd

---

## 📚 变更记录

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-04-06 | 2026.4.2 | 初始版本，记录完整部署流程 |

---

*本文档由 AI 生成并维护，如有问题请提交 PR 更新。*
