<script setup>
import { ref } from 'vue'

const isListening = ref(false)
const transcript = ref('')
const recognition = ref(null)

const startListening = () => {
  if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
    recognition.value = new SpeechRecognition()
    recognition.value.continuous = true
    recognition.value.interimResults = true
    recognition.value.lang = 'zh-CN'

    recognition.value.onresult = (event) => {
      let interimTranscript = ''
      let finalTranscript = ''

      for (let i = event.resultIndex; i < event.results.length; i++) {
        const transcript = event.results[i][0].transcript
        if (event.results[i].isFinal) {
          finalTranscript += transcript + ' '
        } else {
          interimTranscript += transcript
        }
      }
      transcript.value = finalTranscript + interimTranscript
      
      // 如果有最终结果，发送到API
      if (finalTranscript.trim() !== '') {
        sendToAPI(finalTranscript.trim())
      }
    }

    recognition.value.onerror = (event) => {
      console.error('Speech recognition error', event.error)
      stopListening()
    }

    recognition.value.start()
    isListening.value = true
  } else {
    alert('您的浏览器不支持语音识别功能')
  }
}

const stopListening = () => {
  if (recognition.value) {
    recognition.value.stop()
    isListening.value = false
  }
}

const sendToAPI = async (text) => {
  try {
    const payload = {
      session_id: "e9756b0f4e904791a7645be0fa79b867",
      action: "speakMessage",
      text: text
    }

    const response = await fetch('https://jll-cn.app.n8n.cloud/webhook-test/e3ff14eb-db4a-44b2-a574-6d60f564ec4e', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload)
    })

    if (response.ok) {
      console.log('语音文本成功发送到API')
    } else {
      console.error('API请求失败:', response.status)
    }
  } catch (error) {
    console.error('发送到API时出错:', error)
  }
}
</script>

<template>
  <div class="voice-container">
    <div class="voice-input">
      <h1>语音输入</h1>
      <div class="transcript">{{ transcript }}</div>
      <button 
        @click="isListening ? stopListening() : startListening()" 
        :class="{ listening: isListening }"
        class="voice-button"
      >
        {{ isListening ? '停止录音' : '开始录音' }}
      </button>
      <p class="hint">点击按钮开始语音输入</p>
    </div>
  </div>
</template>

<style scoped>
.voice-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.voice-input {
  background: white;
  padding: 2rem;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  text-align: center;
  max-width: 350px;
  width: 90%;
  margin: 0 auto;
}

h1 {
  color: #333;
  margin-bottom: 1.5rem;
  font-size: 2rem;
}

.transcript {
  min-height: 100px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  padding: 1rem;
  margin-bottom: 1.5rem;
  background: #f9f9f9;
  font-size: 1.1rem;
  line-height: 1.6;
}

.voice-button {
  background: #4CAF50;
  color: white;
  border: none;
  padding: 1rem 2rem;
  font-size: 1.2rem;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-bottom: 1rem;
}

.voice-button:hover {
  background: #45a049;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.voice-button.listening {
  background: #f44336;
  animation: pulse 1.5s infinite;
}

.voice-button.listening:hover {
  background: #d32f2f;
}

@keyframes pulse {
  0% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(244, 67, 54, 0.7);
  }
  70% {
    transform: scale(1.05);
    box-shadow: 0 0 0 10px rgba(244, 67, 54, 0);
  }
  100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(244, 67, 54, 0);
  }
}

.hint {
  color: #666;
  font-size: 0.9rem;
  margin: 0;
}
</style>
