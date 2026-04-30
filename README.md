# ch-ai-frontend

一个基于 `Vue 3 + Vite` 的 AI 前端平台，提供两个可切换的智能应用入口：

- `AI 恋爱大师`：面向情感咨询与恋爱沟通场景的对话助手
- `AI 超级智能体`：面向通用问题解决的全能型 AI 助手

## 项目简介

本项目采用单页应用架构，首页作为 AI 应用导航入口，用户可以选择不同的 AI 能力进入对应的聊天页面。

### 主要功能

- 首页应用卡片选择
- 两个独立的 AI 聊天页面
- 支持流式消息展示
- 支持展示 AI 的思考过程
- 自动生成会话 ID
- 返回首页快速切换应用

## 技术栈

- `Vue 3`
- `Vue Router`
- `Vite`
- `Axios`

## 目录结构

```text
src/
├── assets/
│   └── styles/
│       └── global.css
├── router/
│   └── index.js
├── utils/
│   └── api.js
├── views/
│   ├── Home.vue
│   ├── LoveApp.vue
│   └── Manus.vue
├── App.vue
└── main.js
```

## 页面说明

### 首页

首页展示两个 AI 应用入口，点击卡片即可进入对应功能页。

### AI 恋爱大师

- 路由路径：`/love-app`
- 适用于情感问题、恋爱沟通、关系建议等场景
- 通过后端接口 `/api/ai/love_app/chat/sse` 进行流式交互

### AI 超级智能体

- 路由路径：`/manus`
- 适用于更通用的复杂问题咨询与任务辅助
- 通过后端接口 `/api/ai/manus/chat` 获取回复

## 接口配置

项目默认将请求发送到 `/api` 前缀，接口代理或后端服务需要在运行环境中正确配置。

`src/utils/api.js` 中统一配置了 Axios 实例：

```javascript
import axios from 'axios'

const api = axios.create({
  baseURL: '/api',
  timeout: 60000
})
```

## 本地运行

### 安装依赖

```bash
npm install
```

### 启动开发环境

```bash
npm run dev
```

### 打包构建

```bash
npm run build
```

### 预览构建产物

```bash
npm run preview
```

## 运行要求

- Node.js 18+ 推荐
- npm 9+ 推荐
- 后端 AI 服务可用，并且接口路径与前端保持一致

## 使用说明

1. 打开首页
2. 选择需要的 AI 应用
3. 输入问题并发送
4. 查看流式返回内容与思考过程
5. 使用返回按钮切回首页

## 备注

如果后端接口地址与当前默认路径不同，可以在开发代理或接口封装层中进行调整。
