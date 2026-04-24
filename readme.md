# yuko-chat-frontend

yuko-chat 是一个基于 Golang 实现的前后端分离仿微信 IM 系统，本仓库为其前端部分。

后端项目地址：
👉 [https://github.com/Tensort-cat/yuko-chat-backend](https://github.com/Tensort-cat/yuko-chat-backend)

---

## 🚀 技术栈

* Vue
* Vue Router
* Vuex
* Nginx（反向代理 + HTTPS）
* 后端相关：Golang / Gin / Gorm / MySQL / Redis / Kafka

---

## 📦 项目结构

```
yuko-chat-frontend
├── src/
│   ├── assets/        # 静态资源（证书、样式、图片、工具函数）
│   ├── components/    # 通用组件（弹窗、视频、管理等）
│   ├── router/        # 路由配置
│   ├── store/         # Vuex 状态管理
│   ├── views/         # 页面视图
│   │   ├── access/    # 登录 / 注册 / 短信登录
│   │   ├── chat/      # 聊天相关（联系人 / 会话 / 用户信息）
│   │   └── manager/   # 管理后台
│   ├── App.vue
│   └── main.js
├── nginx-1.26.3/      # 本地 nginx（用于 HTTPS + 反向代理）
├── package.json
├── vue.config.js
```

---

## ⚙️ 运行说明

### 1. 修改配置

启动前请根据实际环境修改：

* `src/store/index.js`（后端 API / WebSocket 地址）
* `vue.config.js`（开发环境配置）

---

### 2. 开发模式运行

```bash
yarn install
yarn serve
```

访问：

```
https://your_ip:8080
```

> ⚠️ 需要本地 HTTPS 证书支持，否则 WebSocket（wss）可能无法正常工作

---

### 3. 构建生产包

```bash
yarn build
```

构建结果输出到：

```
/dist
```

---

## 🌐 Nginx 部署（推荐）

注意，项目里这个 nginx 是 Windows 版本的，如果是其他 OS 请自行下载替换。

### 1. 证书准备

请自行生成 SSL 证书，并放入：

```
src/assets/cert/
```

例如：

```
192.168.3.24+2.pem
192.168.3.24+2-key.pem
```

---

### 2. Nginx 配置示例

```nginx
# xxx 为你本地的绝对路径前缀
server {
    listen 443 ssl;
    server_name 192.168.3.24;

    ssl_certificate xxx/yuko-chat-front/src/assets/cert/192.168.3.24+2.pem;
    ssl_certificate_key xxx/yuko-chat-front/src/assets/cert/192.168.3.24+2-key.pem;

    # 前端
    location / {
        root xxx/yuko-chat-front/dist;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # 后端 API
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
    }

    # WebSocket
    location /wss {
        proxy_pass http://127.0.0.1:8000;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }

    # 静态资源代理
    location /static/ {
        proxy_pass http://127.0.0.1:8000;
    }
}
```

---

### 3. 启动 Nginx

进入目录：

```
nginx-1.26.3/
```

启动：

```bash
start nginx
```

访问：

```
https://your_ip
```

---

### 4. 关闭 Nginx

```bash
nginx -s quit
```

---

## ✨ 功能概述

* 用户登录 / 注册 / 邮箱验证码登录
* 好友 / 群组管理
* 单聊 / 群聊
* 会话列表
* WebSocket 实时通信
* 基于 WebRTC 的一对一视频通话
* 后台管理页面

---

## 📌 注意事项

* 必须使用 HTTPS（否则 WebSocket wss 无法使用）
* 前后端地址需保持一致（协议 + 域名）
* 请勿提交真实证书或敏感配置到仓库
* 建议使用 Nginx 统一代理 API 与 WebSocket