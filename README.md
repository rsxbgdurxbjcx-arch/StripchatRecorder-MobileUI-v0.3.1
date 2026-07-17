# StripchatRecorder-MobileUI-v0.3.1

[简体中文](README.md) | [English](README.en.md)

自托管的 Stripchat 直播录制工具，提供基于 Web 的管理界面，支持自动录制、后处理流水线和多渠道通知。

> 本项目 Fork 自 [ChanTrail/StripchatRecorder](https://github.com/ChanTrail/StripchatRecorder)，在原版基础上新增了：
> - **全新 UI 设计**：暖调 Noir Crimson 主题（深/浅色随系统自动切换）、绯红主色、细腻投影与按压反馈、毛玻璃导航
> - **安卓移动端全面适配**：底部导航栏（带图标 + 安全区适配）、主播卡片 2 列极致紧凑布局、录制表格独立滚动 + 图标化操作列、固定页面大小
> - **登录系统**：初始账号 `sr-mobileui` / 密码 `admin`，支持在设置中修改账号密码
> - **Telegram 上传后自动删除本地文件**：上传成功后自动清理本地视频

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-3.0.html)

---

## 功能特性

- 监控多个主播，上线时自动开始录制
- Web UI 管理主播、录制文件和后处理任务
- **安卓移动端全面适配**：底部导航栏（带图标）、主播卡片 2 列、录制表格紧凑、提示反馈不遮挡菜单
- **登录系统**：基于 token 的认证，保护 Web UI 不被未授权访问
- **主播查找**：通过 [camgirlfinder.net](https://camgirlfinder.net) 查找主播
- **转发流（HLS Relay）**：无需录制即可将主播直播流转发给播放器
- 支持分离式网络代理：可分别配置 Stripchat API 代理与 CDN 分片代理
- **Mouflon HLS 解密**：支持管理 `pkey → pdkey` 密钥对
- 可配置的后处理流水线，支持插件化模块：
  - **contact_sheet** — 生成带时间戳的缩略图预览图
  - **filter_short** — 删除低于最短时长的录制文件
  - **notify_discord** — 通过 Discord Webhook 发送录制信息和封面图
  - **notify_telegram** — 通过 MTProto 发送录制信息、封面图和视频（**支持上传后自动删除本地文件**）
- 录制文件页磁盘空间监控
- 基于 SSE 的实时 UI 更新，支持多客户端同步
- 跟随系统主题的深色/浅色模式

---

## 快速开始（Docker）

### 部署方式

```bash
git clone https://github.com/rsxbgdurxbjcx-arch/StripchatRecorder-MobileUI-v0.3.1.git
cd StripchatRecorder-MobileUI-v0.3.1
docker compose up -d
```

启动后在浏览器中打开 `http://localhost:4040`，使用初始账号 `sr-mobileui` / 密码 `admin` 登录。

> **部署速度说明**：本项目拉取 Docker Hub 上的预构建镜像 `chantrail/stripchat-recorder:latest` 提供运行环境，通过 volume mount 注入自定义后端二进制（含移动端适配 + 登录系统）和修改后的 `notify_telegram` 模块二进制。整个部署过程通常在 **2 分钟内**完成。

Docker 镜像以 Server 模式运行（端口 4040），配置写入挂载的 `config/settings.json`。

### 主要设置项

在 Web UI 的「设置」页面可配置以下选项：

| 设置项 | 说明 |
|--------|------|
| 输出目录 | 录制文件保存路径 |
| 最大并发录制数 | 同时录制的最大主播数，`0` 表示不限制 |
| 轮询间隔 | 检查主播是否上线的间隔（秒），范围 10–300 |
| 合并格式 | 录制结束后自动合并分片的格式：`mp4`（默认）、`mkv`、`ts` |
| 上线自动录制 | 新添加的主播是否默认开启自动录制 |
| 后处理临时目录最大占用 | 后处理模块运行时产生的临时文件上限（GB），超出后自动删除最旧的文件，`0` 表示不限制，默认 50 GB |

### 网络代理与镜像站

在设置页的「网络」中可分别配置 API 代理、CDN 代理和 Stripchat 镜像站。

### Mouflon HLS 解密密钥

Stripchat 对 HLS 分片文件名进行了加密（Mouflon 系统）。若录制时遇到无法下载分片的情况，需在设置页的「Mouflon 解密密钥」中填入对应的 `pkey → pdkey` 密钥对。

### 转发流（HLS Relay）

在 Server 模式下，直接用播放器打开以下地址即可播放直播：

```
http://localhost:4040/stream/{modelname}
```

---

## 新增功能详解

### 1. 安卓移动端全面适配

- **底部导航栏**：移动端（< 768px）自动隐藏桌面侧边栏，显示底部导航栏，每个菜单带功能图标
- **主播卡片 2 列**：移动端主播列表一行展示 2 个卡片，卡片高度紧凑，缩略图完整显示
- **录制表格紧凑**：移动端录制文件表格缩小字体、减小间距，适配窄屏
- **提示反馈不遮挡**：toast 提示上移，不遮挡底部导航栏
- **固定页面大小**：使用 `position: fixed` 固定 viewport，避免安卓浏览器地址栏伸缩导致布局跳动
- **输入框防缩放**：移动端输入框字体 16px，防止安卓 Chrome 聚焦缩放
- **后处理输入框对齐**：所有模块的输入框统一高度对齐

### 2. 登录系统

- **初始账号**：`sr-mobileui` / 密码：`admin`
- **认证机制**：基于 token 的认证，登录后所有 API 请求携带 `Authorization: Bearer {token}`
- **SSE 认证**：SSE 连接通过查询参数传递 token（EventSource 不支持自定义 header）
- **修改密码**：在「设置」页面底部可修改账号和密码，修改后自动退出重新登录
- **持久化**：账号密码存储在 `config/auth.json`（密码用 SHA-256 哈希）
- **安全**：修改密码后所有已登录 token 失效，强制重新登录

### 3. Telegram 上传后自动删除

在 Web UI 的「后处理」页面，编辑 `notify_telegram` 节点参数时，可看到「**上传后自动删除本地文件**」开关：

- **关闭（默认）**：上传成功后保留本地视频文件
- **开启**：视频文件上传 Telegram 成功后，立即自动删除本地视频文件及其元数据

---

## 后处理模块

模块是实现了简单协议的独立可执行文件。仓库 `data/modules/notify_telegram_v030` 已包含修改后的预编译二进制。其余三个模块由 Docker 镜像内置提供。

---

## 技术栈

- **前端：** Vue 3, TypeScript, Vite, Tailwind CSS, Reka UI
- **后端：** Rust, Axum
- **后处理模块：** Rust（独立二进制）
- **容器：** Debian, ffmpeg

---

## 致谢

本项目基于 [ChanTrail/StripchatRecorder](https://github.com/ChanTrail/StripchatRecorder) 开发，感谢原作者的贡献。

---

## 开源许可证

本项目基于 [GNU 通用公共许可证 v3.0](https://www.gnu.org/licenses/old-licenses/gpl-3.0.html) 发布。

---

## 免责声明

本项目仅用于技术研究与学习交流。使用者需自行承担部署、运维与合规风险。
