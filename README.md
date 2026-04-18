# 云朵机 ☁️

一个模拟手机界面的 AI 角色扮演聊天应用，以单一 HTML 文件形式运行，支持 PWA 安装，无需后端服务器。

---

## 目录

- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [项目结构](#项目结构)
- [依赖说明](#依赖说明)
- [核心模块](#核心模块)
- [数据存储](#数据存储)
- [配置说明](#配置说明)
- [PWA 支持](#pwa-支持)
- [注意事项](#注意事项)

---

## 功能特性

### 💬 AI 角色聊天
- 支持创建多个 AI 角色，自定义名字、头像、性格设定、系统提示词
- 支持单人聊天和群聊模式
- 聊天气泡主题可切换（默认紫、粉蓝、暗紫星空、橙暖秋、墨绿极简、黑白简约）
- 支持发送图片、语音气泡、表情包（贴纸）
- 支持 AI 角色主动发送表情包（emojiMatch 模式）
- 支持 AI 视频通话模拟（需配置 API Key）
- 支持正则渲染器，对 AI 回复内容进行自定义查找/替换处理

### 🎵 音乐播放器
- Dynamic Island 黑胶唱片风格播放控件
- 支持 Jamendo 等免费音乐来源

### 📱 模拟手机界面
- 仿 iOS 风格主屏幕，带动态岛、状态栏、底部 Dock
- 应用包含：微信（聊天）、今日、论坛、音乐、收藏、相册等模块
- 深色模式支持

### 🌐 实时天气
- 接入 Open-Meteo API，将当地天气信息注入 AI 提示词

### 🔗 多设备同步（Beta）
- 通过 PeerJS 实现 P2P 设备连接
- 支持将聊天记录导出/导入同步到另一台设备

### ✍️ 其他
- 今日页：待办事项（Todo）、日历、今日壁纸
- 论坛页：AI 角色模拟发帖互动
- 收藏页：收藏重要聊天消息
- API 调用记录查看器
- 角色记忆管理（Dexie.js IndexedDB 存储）

---

## 快速开始

1. 将 `index.html` 和配套的 `manifest.json`、`sw.js` 放在同一目录下（或直接单文件运行）。
2. 用浏览器打开 `index.html`。
3. 进入「设置 → API 配置」填入你的 AI API Key（兼容 OpenAI 格式接口）。
4. 在「角色」模块创建第一个 AI 角色，开始聊天。

> 无需 Node.js 或任何构建工具，直接双击打开即可使用。

---

## 项目结构

```
.
├── index.html        # 全部应用代码（HTML + CSS + JS，约 25000 行）
├── manifest.json     # PWA 清单文件
└── sw.js             # Service Worker（离线缓存）
```

整个应用为单文件架构，所有 CSS 样式、JavaScript 逻辑、HTML 模板均内联在 `index.html` 中。

---

## 依赖说明

所有依赖均通过 CDN 加载，无需本地安装：

| 依赖 | 用途 |
|------|------|
| [Noto Sans SC](https://fonts.google.com/noto/specimen/Noto+Sans+SC) | 中文字体 |
| [pako](https://cdnjs.cloudflare.com/ajax/libs/pako/2.1.0/) | 数据压缩/解压 |
| [Dexie.js](https://unpkg.com/dexie@3/) | IndexedDB 封装，用于角色记忆存储 |
| [PeerJS](https://unpkg.com/peerjs) | P2P 设备连接（按需加载） |
| [Open-Meteo API](https://api.open-meteo.com/) | 免费天气数据（无需 Key） |

---

## 核心模块

### 状态管理 (`S` 对象)
全局状态通过 `window.S` 统一管理，包含：
- `S.characters` — 角色列表
- `S.chats` — 各角色聊天记录
- `S.settings` — 全局设置（API Key、深色模式、字体等）
- `S.chatSettings` — 各角色独立配置（昵称、头像、记忆等）
- `S.stickerPack` — 表情包列表
- `S.moments` — 朋友圈动态
- `S.apiHistory` — API 调用日志

### 页面路由
通过 `showScreen(screenId)` 函数切换各功能页面：

| Screen ID | 对应页面 |
|-----------|----------|
| `screen-home` | 主屏幕 |
| `screen-wechat` | 聊天列表 |
| `chat-screen` | 聊天界面 |
| `screen-settings` | 设置 |
| `screen-characters` | 角色管理 |
| `screen-music` | 音乐播放器 |
| `screen-favorites` | 消息收藏 |
| `screen-renderer` | 正则渲染器 |
| `screen-api-history` | API 调用记录 |

### AI 接口
支持兼容 OpenAI Chat Completions 格式的任意接口，在设置中可配置：
- API Key
- API Base URL（默认 `https://api.openai.com/v1`）
- 模型名称
- 温度（temperature）
- 最大 Token 数

### 正则渲染器
在「渲染器」应用中可添加多条正则替换规则，所有 AI 回复消息在渲染时会按顺序应用这些规则，支持 HTML 标签替换（如加粗、变色等）。

---

## 数据存储

所有数据存储在浏览器本地，**不上传至任何服务器**：

| 存储位置 | 内容 |
|----------|------|
| `localStorage` | 聊天记录、角色、设置、表情包、收藏、朋友圈、待办等 |
| `IndexedDB`（Dexie） | 角色长期记忆条目 |

> ⚠️ 清除浏览器数据会导致所有本地记录丢失，建议定期通过「设置 → 导出数据」备份。

---

## 配置说明

### 全局设置
进入主屏幕 → 「设置」图标：
- **API Key / Base URL / 模型**：AI 接口配置
- **深色模式**：切换深色/浅色主题
- **字体风格**：默认 / 衬线 / 等宽 / 圆体
- **气泡主题**：全局聊天气泡颜色方案
- **用户名 / 头像**：自定义自己的显示名称与头像

### 角色配置
进入「角色」→ 新建或编辑角色：
- 名字、头像、简介
- 系统提示词（人设）
- 记忆功能开关
- 表情包发送模式（关闭 / 匹配发送 / AI 识别）
- 群聊成员配置

---

## PWA 支持

应用支持作为 PWA 安装到手机桌面：
1. 需要通过 HTTPS 或 `localhost` 访问。
2. 同目录需存在 `manifest.json` 和 `sw.js`。
3. 浏览器会提示「添加到主屏幕」。

安装后支持离线访问（Service Worker 缓存），并会在检测到新版本时提示刷新。

---

## 注意事项

- 本应用需要你自行提供 AI API Key，产生的 API 调用费用由用户自行承担。
- 所有聊天数据仅存储在本地浏览器，开发者不收集任何数据。
- 视频通话为模拟 UI，实际调用 AI 接口生成文字对话，并非真实音视频通话。
- P2P 同步功能依赖 PeerJS 公共服务器进行信令，网络条件不佳时可能不稳定。
- 建议在现代浏览器（Chrome / Safari / Firefox 最新版）中使用以获得最佳体验。
