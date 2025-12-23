# E视界 (DongguaTV Enhanced Edition)

这是一个经过全面重构和升级的现代流媒体聚合播放器，基于 Node.js 和 Vue 3 构建。相比原版，本作引入了 Netflix 风格的沉浸式 UI、TMDb 数据驱动的动态榜单、以及智能的多源聚合搜索功能。

---

## 📚 目录 (Table of Contents)

- [✨ 核心特性 (Features)](#-核心特性-features)
- [🎨 界面升级 (UI Upgrade)](#-界面升级-ui-upgrade)
- [🚀 快速开始 (Quick Start)](#-快速开始-quick-start)
  - [使用 Docker (推荐)](#-docker-部署-推荐)
  - [一键安装脚本](#-一键安装脚本-linux)
  - [Vercel 部署](#-vercel-部署)
- [⚙️ 核心配置 (Configuration)](#️-核心配置-configuration)
  - [配置采集源 (db.json)](#1-配置采集源-dbjson-重要)
  - [配置 API Key (.env)](#2-配置-api-key-env-必需)
- [🌐 网络优化 & 代理 (Network & Proxy)](#-网络优化--代理-network--proxy)
  - [TMDB 代理 (解决海报加载)](#1-tmdb-代理-解决海报加载)
  - [CORS 视频代理 (解决播放失败)](#2-cors-视频代理-解决播放失败)
- [🔒 高级功能 (Advanced)](#-高级功能-advanced)
  - [访问密码](#1-全局访问密码)
  - [多用户同步](#2-多用户历史同步)
  - [远程配置](#3-远程配置文件)
- [📱 移动端 (Mobile App)](#-移动端-mobile-app)
- [⚠️ 免责声明 (Disclaimer)](#️-免责声明-disclaimer)

---

## ✨ 核心特性 (Features)

| 特性 | 说明 |
|------|------|
| **🎬 双引擎驱动** | **TMDb** (元数据) + **Maccms** (资源)，数据质量与播放速度兼得。 |
| **🔍 智能聚合** | **实时流式搜索 (SSE)**，边搜边显；自动聚合同一影片的不同线路。 |
| **⚡ 极速缓存** | 内置 **SQLite** 数据库缓存，热搜词秒级响应，支持无限存储。 |
| **📺 沉浸播放** | **影院模式**设计，智能线路测速，自动故障转移，支持 DLNA/AirPlay 投屏。 |
| **🌏 大陆优化** | 智能 IP 检测，自动切换 **TMDB 反代**；内置 **CORS 视频代理**，解决资源站防盗链。 |
| **📱 多端适配** | 完美适配 **Android TV** (遥控器)、**手机** (刘海屏) 和 **PC**。 |

---

## 🎨 界面升级 (UI Upgrade)

| 区域 | 原版体验 | **MAX 版体验** |
| :--- | :--- | :--- |
| **首页** | 简单列表 | **Netflix 风格 Hero 轮播**：全屏动态背景、Top 10 排名特效。 |
| **导航** | 固定顶部 | **智能融合导航**：初始透明，滚动变黑；支持平滑滚动定位。 |
| **搜索** | 加载等待 | **实时流式加载 (SSE)**：结果即时呈现，源数量实时跳动增加。 |
| **线路** | 单一延迟 | **双模式测速**：区分"直连"(客户端)和"代理"(服务端) 测速。 |

---

## 🚀 快速开始 (Quick Start)

### 🐳 Docker 部署 (推荐)

最简单的方式，支持 x86 和 ARM (M1/M2/树莓派)。

```bash
# 1. 创建必要文件 (防止挂载成目录)
touch db.json cache.db
echo '{"sites":[]}' > db.json

# 2. 启动容器
docker run -d -p 3000:3000 \
  -e TMDB_API_KEY="your_api_key_here" \
  -e ACCESS_PASSWORD="your_password" \
  -v $(pwd)/db.json:/app/db.json \
  -v $(pwd)/cache.db:/app/cache.db \
  --name donggua-tv \
  --restart unless-stopped \
  ghcr.io/ednovas/dongguatv:latest
```

### 📜 一键安装脚本 (Linux)

适用于 Ubuntu/Debian/CentOS。

```bash
curl -fsSL https://raw.githubusercontent.com/ednovas/dongguaTV/main/install.sh | bash
```
脚本会引导您配置 API Key、端口等信息。

### ▲ Vercel 部署

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fednovas%2FdongguaTV&env=TMDB_API_KEY,REMOTE_DB_URL,ACCESS_PASSWORD,TMDB_PROXY_URL)

> **注意**：Vercel 部署必须配置 `REMOTE_DB_URL` (远程配置)，因为无法修改本地 `db.json`。

---

## ⚙️ 核心配置 (Configuration)

### 1. 配置采集源 (`db.json`) ⚠️ 重要

项目**不包含**内置资源，您必须配置 Maccms V10 接口才能使用。
项目启动后，请编辑根目录下的 `db.json`：

```json
{
  "sites": [
    {
      "key": "unique_id_1",
      "name": "站点名称",
      "api": "https://example.com/api.php/provide/vod/",
      "active": true
    }
  ]
}
```

### 2. 配置 API Key (`.env`) 必需

必需注册 **The Movie Database (TMDb)** 获取 API Key。
在 `.env` 文件中配置：

```env
TMDB_API_KEY=your_key_here
```

---

## 🌐 网络优化 & 代理 (Network & Proxy)

### 1. TMDB 代理 (解决海报加载)

大陆服务器无法直接访问 TMDB API 和图片，需要配置反代。

**部署方法 (Cloudflare Workers):**
1. 登录 Cloudflare -> Workers，创建新 Worker。
2. 粘贴 `cloudflare-tmdb-proxy.js` 代码并部署。
3. 在 `.env` 中配置：
   ```env
   TMDB_PROXY_URL=https://your-worker.workers.dev
   ```

### 2. CORS 视频代理 (解决播放失败)

当视频流有 CORS 限制或直连太慢时，系统自动使用代理。支持 **m3u8 重写** 和 **慢速检测**。

**部署方法 (Cloudflare Workers - 推荐):**
1. 创建新 Worker，粘贴 `cloudflare-cors-proxy.js` **全部代码**。
2. 在 `.env` 中配置：
   ```env
   CORS_PROXY_URL=https://your-cors-worker.workers.dev
   ```

**部署方法 (VPS / Node.js):**
如果不希望受限于 Cloudflare 每日请求限额，可用 VPS 运行：
1. 上传 `proxy-server.js` 到 VPS。
2. 运行 `PORT=8080 node proxy-server.js`。
3. 配置 `.env`: `CORS_PROXY_URL=http://your-vps-ip:8080`。

---

## 🔒 高级功能 (Advanced)

### 1. 全局访问密码
在 `.env` 中设置，开启后必须输入密码才能访问页面。
```env
ACCESS_PASSWORD=my_secret_pass
```

### 2. 多用户历史同步 (SQLite 模式)
设置多个密码，逗号分隔。每个密码对应一个独立账户，支持多设备同步播放进度。
```env
# 第一个密码为本地模式(管理员)，后续密码为同步模式(家庭成员)
ACCESS_PASSWORD=admin,user1,user2
CACHE_TYPE=sqlite
```

### 3. 远程配置文件
从 URL 加载 `db.json`，方便多站点统一管理。
```env
REMOTE_DB_URL=https://example.com/config/db.json
```

---

## 📱 移动端 (Mobile App)

本项目支持构建 Android APK，适配电视遙控器和手机。

1. **自动构建**：Fork 本项目，推送 Tag (如 `v1.0.0`)，GitHub Actions 会自动构建并发布 APK。
2. **自定义地址**：修改 `capacitor.config.json` 中的 `server.url` 为您的服务器地址。

---

## ⚠️ 免责声明 (Disclaimer)

1.  **仅供学习**：本项目仅作为技术学习用途，旨在展示 Node.js/Vue 3 开发技术。
2.  **内容无关**：本项目**不内置**任何资源接口。开发者不对用户配置的任何内容负责。
3.  **合规使用**：请使用者自行遵守当地法律法规，仅在合法范围内使用。

*Enjoy your movie night! 🍿*
