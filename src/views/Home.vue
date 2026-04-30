<template>
  <div class="home">
    <div class="container">
      <h1 class="title">AI 应用平台</h1>
      <p class="subtitle">选择您想要使用的 AI 应用</p>
      <div class="app-list">
        <div class="app-card" @click="goToApp('/love-app')">
          <div class="app-icon love-icon">💕</div>
          <h2>AI 恋爱大师</h2>
          <p>智能情感对话助手，助您解决情感问题</p>
        </div>
        <div class="app-card" @click="goToApp('/manus')">
          <div class="app-icon manus-icon">🤖</div>
          <h2>AI 超级智能体</h2>
          <p>全能型 AI 助手，解决各类复杂问题</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

const goToApp = (path) => {
  router.push(path)
}
</script>

<style scoped>
.home {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.container {
  text-align: center;
  padding: 40px;
}

.title {
  font-size: 48px;
  color: white;
  margin-bottom: 16px;
  font-weight: 700;
}

.subtitle {
  font-size: 20px;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 48px;
}

.app-list {
  display: flex;
  gap: 32px;
  justify-content: center;
}

.app-card {
  width: 300px;
  padding: 40px;
  background: white;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
}

.app-card:hover {
  transform: translateY(-10px);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
}

.app-icon {
  font-size: 64px;
  margin-bottom: 20px;
}

.app-card h2 {
  font-size: 24px;
  color: #333;
  margin-bottom: 12px;
}

.app-card p {
  font-size: 14px;
  color: #666;
  line-height: 1.6;
}
</style>
