<template>
  <div class="chat-container">
    <div class="chat-header">
      <div class="header-content">
        <button class="back-btn" @click="goHome">← 返回</button>
        <h1>🤖 AI 超级智能体</h1>
        <div class="chat-id">会话ID: {{ chatId }}</div>
      </div>
    </div>

    <div class="chat-messages" ref="messagesContainer">
      <div v-if="messages.length === 0" class="welcome-tip">
        <p>👋 欢迎使用 AI 超级智能体！</p>
        <p>我是您的全能AI助手，可以帮助您解决各类复杂问题。</p>
      </div>

      <div
        v-for="(msg, index) in messages"
        :key="index"
        :class="['message', msg.role === 'user' ? 'user-message' : 'ai-message']"
      >
        <div class="avatar">{{ msg.role === 'user' ? '👤' : '🤖' }}</div>
        <div class="message-content">
          <div class="message-text" v-html="msg.content"></div>
          <div v-if="msg.thought" class="thought-container">
            <div class="thought-header" @click="toggleThought(index)">
              <span class="thought-icon">{{ msg.thoughtExpanded ? '▼' : '▶' }}</span>
              <span>思考过程</span>
            </div>
            <div v-if="msg.thoughtExpanded" class="thought-content" v-html="msg.thought"></div>
          </div>
          <div class="message-time">{{ msg.time }}</div>
        </div>
      </div>

      <div v-if="isLoading" class="message ai-message">
        <div class="avatar">🤖</div>
        <div class="message-content">
          <div class="message-text loading">
            <span class="loading-dot">.</span>
            <span class="loading-dot">.</span>
            <span class="loading-dot">.</span>
          </div>
        </div>
      </div>
    </div>

    <div class="chat-input-area">
      <div class="input-wrapper">
        <textarea
          v-model="inputMessage"
          @keydown.enter.exact="sendMessage"
          placeholder="输入您的问题..."
          rows="1"
        ></textarea>
        <button class="send-btn" @click="sendMessage" :disabled="!inputMessage.trim() || isLoading">
          发送
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const messagesContainer = ref(null)
const messages = ref([])
const inputMessage = ref('')
const isLoading = ref(false)
const chatId = ref('')
let sseEmitter = null

onMounted(() => {
  chatId.value = generateChatId()
})

const generateChatId = () => {
  return 'manus_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9)
}

const goHome = () => {
  if (sseEmitter) {
    sseEmitter.close()
  }
  router.push('/')
}

const scrollToBottom = () => {
  nextTick(() => {
    if (messagesContainer.value) {
      messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
    }
  })
}

const toggleThought = (index) => {
  messages.value[index].thoughtExpanded = !messages.value[index].thoughtExpanded
  scrollToBottom()
}

const sendMessage = async () => {
  const message = inputMessage.value.trim()
  if (!message || isLoading.value) return

  messages.value.push({
    role: 'user',
    content: message,
    time: new Date().toLocaleTimeString()
  })
  inputMessage.value = ''
  isLoading.value = true
  scrollToBottom()

  const aiMsgIndex = messages.value.length
  messages.value.push({
    role: 'ai',
    content: '',
    thought: '',
    thoughtExpanded: false,
    time: new Date().toLocaleTimeString()
  })

  // 构建消息列表数据
  const messageList = messages.value.map(msg => ({
    role: msg.role,
    content: msg.content
  }))
  
  console.log("---Sending messageList---", messageList)

  // 使用GET请求发送数据
  try {
    const response = await fetch(`/api/ai/manus/chat?message=${encodeURIComponent(message)}`)

    if (!response.ok) {
      throw new Error('Network response was not ok')
    }

    const reader = response.body.getReader()
    const decoder = new TextDecoder()

    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      const chunk = decoder.decode(value)
      console.log("---Received chunk---", chunk)

      // 处理SSE格式
      const lines = chunk
        .split('\n')
        .map(line => line.replace(/^data:/, '').trim())
        .filter(line => line)

      lines.forEach(line => {
        try {
          const data = JSON.parse(line)
          console.log("---Parsed data---", data)

          if (data.content) {
            // 区分思考消息和真正的回复
            if (data.content.includes('思考完成') || data.content.includes('Step ')) {
              // 这是思考过程
              messages.value[aiMsgIndex].thought += data.content + '<br>'
            } else {
              // 这是真正的回复
              messages.value[aiMsgIndex].content += data.content
            }
          } else if (data.messageList) {
            // 处理后端发送的消息列表
            console.log("---Received messageList---", data.messageList)
            // 清空当前AI消息
            messages.value[aiMsgIndex].content = ''
            // 从消息列表中提取AI的最终回复
            const aiMessages = data.messageList.filter(msg => msg.role === 'assistant' || msg.role === 'AI' || msg.messageType === 'ASSISTANT')
            if (aiMessages.length > 0) {
              const finalResponse = aiMessages[aiMessages.length - 1].content || aiMessages[aiMessages.length - 1].text
              // 处理格式
              const processedResponse = finalResponse
                .replace(/\n/g, '<br>')
                .replace(/^\s*•\s*/gm, '<span style="display: block; margin-left: 20px;">• </span>')
                .replace(/^\s*✓\s*/gm, '<span style="display: block; margin-left: 20px;">✓ </span>')
              messages.value[aiMsgIndex].content = processedResponse
            }
          } else if (Array.isArray(data)) {
            // 处理直接返回的消息数组
            console.log("---Received message array---", data)
            // 清空当前AI消息
            messages.value[aiMsgIndex].content = ''
            // 从消息数组中提取AI的最终回复
            const aiMessages = data.filter(msg => msg.role === 'assistant' || msg.role === 'AI' || msg.messageType === 'ASSISTANT')
            if (aiMessages.length > 0) {
              const finalResponse = aiMessages[aiMessages.length - 1].content || aiMessages[aiMessages.length - 1].text
              // 处理格式
              const processedResponse = finalResponse
                .replace(/\n/g, '<br>')
                .replace(/^\s*•\s*/gm, '<span style="display: block; margin-left: 20px;">• </span>')
                .replace(/^\s*✓\s*/gm, '<span style="display: block; margin-left: 20px;">✓ </span>')
              messages.value[aiMsgIndex].content = processedResponse
            }
          }
        } catch (e) {
          // 处理非JSON格式的SSE数据
          if (line.includes('思考完成') || line.includes('Step ')) {
            // 这是思考过程
            messages.value[aiMsgIndex].thought += line + '<br>'
          } else {
            // 这是真正的回复
            const processedLine = line
              .replace(/\n/g, '<br>')
              .replace(/^\s*•\s*/gm, '<span style="display: block; margin-left: 20px;">• </span>')
              .replace(/^\s*✓\s*/gm, '<span style="display: block; margin-left: 20px;">✓ </span>')
            messages.value[aiMsgIndex].content += processedLine
          }
        }
        scrollToBottom()
      })
    }
  } catch (error) {
    console.error('Error:', error)
    messages.value[aiMsgIndex].content = '抱歉，发生了错误，请稍后重试。'
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
.chat-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f0f2f5;
}

.chat-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 16px 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.header-content {
  display: flex;
  align-items: center;
  gap: 16px;
}

.back-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
}

.back-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.header-content h1 {
  font-size: 24px;
  margin: 0;
}

.chat-id {
  margin-left: auto;
  font-size: 12px;
  background: rgba(255, 255, 255, 0.2);
  padding: 4px 12px;
  border-radius: 12px;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.welcome-tip {
  text-align: center;
  color: #666;
  padding: 40px;
  font-size: 16px;
}

.welcome-tip p {
  margin: 8px 0;
}

.message {
  display: flex;
  gap: 12px;
  max-width: 70%;
}

.user-message {
  align-self: flex-end;
  flex-direction: row-reverse;
}

.ai-message {
  align-self: flex-start;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #e0e0e0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  flex-shrink: 0;
}

.user-message .avatar {
  background: #2196f3;
}

.ai-message .avatar {
  background: #667eea;
}

.message-content {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.message-text {
  padding: 12px 16px;
  border-radius: 16px;
  line-height: 1.5;
  word-break: break-word;
}

.user-message .message-text {
  background: #2196f3;
  color: white;
  border-bottom-right-radius: 4px;
}

.ai-message .message-text {
  background: white;
  color: #333;
  border-bottom-left-radius: 4px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.message-time {
  font-size: 11px;
  color: #999;
}

.user-message .message-time {
  text-align: right;
}

.thought-container {
  margin-top: 8px;
  border-top: 1px solid #f0f0f0;
  padding-top: 8px;
}

.thought-header {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 12px;
  color: #666;
  padding: 4px 0;
}

.thought-icon {
  font-size: 10px;
  transition: transform 0.3s;
}

.thought-content {
  margin-top: 8px;
  padding: 12px;
  background: #f9f9f9;
  border-radius: 8px;
  font-size: 13px;
  line-height: 1.4;
  color: #555;
  border-left: 3px solid #667eea;
}

.loading {
  display: flex;
  gap: 4px;
}

.loading-dot {
  animation: bounce 1.4s infinite ease-in-out;
}

.loading-dot:nth-child(1) { animation-delay: 0s; }
.loading-dot:nth-child(2) { animation-delay: 0.2s; }
.loading-dot:nth-child(3) { animation-delay: 0.4s; }

@keyframes bounce {
  0%, 80%, 100% { opacity: 0; }
  40% { opacity: 1; }
}

.chat-input-area {
  background: white;
  padding: 16px 24px;
  border-top: 1px solid #e0e0e0;
}

.input-wrapper {
  display: flex;
  gap: 12px;
  align-items: flex-end;
}

.input-wrapper textarea {
  flex: 1;
  padding: 12px 16px;
  border: 1px solid #e0e0e0;
  border-radius: 24px;
  resize: none;
  font-size: 14px;
  outline: none;
  transition: border-color 0.3s;
  font-family: inherit;
}

.input-wrapper textarea:focus {
  border-color: #667eea;
}

.send-btn {
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 24px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.3s;
}

.send-btn:hover:not(:disabled) {
  transform: scale(1.05);
}

.send-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
