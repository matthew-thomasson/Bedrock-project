<script setup>
import { ref, reactive, nextTick, onMounted } from 'vue'
import ChatHeader from './components/ChatHeader.vue'
import ChatMessage from './components/ChatMessage.vue'
import ChatInput from './components/ChatInput.vue'

// State
const messages = ref([])
const inputMessage = ref('')
const isLoading = ref(false)
const isConnected = ref(false)
const errorMessage = ref('')
const selectedModel = ref('anthropic.claude-3-5-sonnet-20241022-v2:0')
const messagesContainer = ref(null)
const messageInput = ref(null)

// API Configuration
const apiConfig = reactive({
    baseUrl: '/api', // Will use backend API endpoints
    region: 'us-east-1'
})

// Initialize connection check
const initializeAWS = async () => {
    try {
        console.log('Checking backend API connectivity...')
        
        // Test backend API connection
        const response = await fetch(`${apiConfig.baseUrl}/health`)
        
        if (response.ok) {
            isConnected.value = true
            console.log('Backend API connected successfully')
        } else {
            throw new Error('Backend API health check failed')
        }
        
    } catch (error) {
        console.error('Failed to connect to backend API:', error)
        errorMessage.value = 'Backend API not available - using demo mode'
        // Fallback to demo mode
        setTimeout(() => {
            isConnected.value = true
        }, 1000)
    }
}

// Send message to backend API
const callBedrock = async (messages, modelId) => {
    try {
        console.log('Sending request to backend API:', { modelId, messageCount: messages.length })
        
        const response = await fetch(`${apiConfig.baseUrl}/chat`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                messages: messages,
                modelId: modelId,
                temperature: 0.7,
                maxTokens: 4000
            })
        })
        
        if (!response.ok) {
            throw new Error(`API request failed: ${response.status} ${response.statusText}`)
        }
        
        const data = await response.json()
        console.log('Received response from backend API')
        
        return data.response || data.content || 'No response from API'
        
    } catch (error) {
        console.error('Backend API call failed:', error)
        
        // Fallback to demo responses if backend is not available
        console.log('Falling back to demo mode...')
        return await callBedrockDemo(messages, modelId)
    }
}

// Demo fallback when backend is not available
const callBedrockDemo = async (messages, modelId) => {
    return new Promise((resolve) => {
        setTimeout(() => {
            const responses = [
                "I'm running in demo mode. To use real AI responses, you'll need to set up the backend API with proper AWS Bedrock integration.",
                "This is a simulated response. The app is designed to work with a secure backend API that handles AWS credentials safely.",
                "Hello! I'm in demo mode since the backend API isn't available. In production, I would connect securely to AWS Bedrock through your backend.",
                "Great question! Once you set up the backend API with proper AWS credentials, I'll be able to provide real AI responses.",
                `I would use the ${modelId} model to answer this, but I'm currently in demo mode for security reasons.`
            ]
            
            const randomResponse = responses[Math.floor(Math.random() * responses.length)]
            resolve(randomResponse)
        }, 1500 + Math.random() * 1000)
    })
}

// Send message
const sendMessage = async () => {
    if (!inputMessage.value.trim() || isLoading.value) return

    const userMessage = {
        id: Date.now(),
        role: 'user',
        content: inputMessage.value.trim(),
        timestamp: new Date()
    }

    messages.value.push(userMessage)
    const currentInput = inputMessage.value
    inputMessage.value = ''
    isLoading.value = true
    errorMessage.value = ''

    await nextTick()
    scrollToBottom()

    try {
        // Prepare conversation history for Bedrock
        const conversationHistory = messages.value
            .filter(msg => msg.role !== 'system')
            .map(msg => ({
                role: msg.role === 'user' ? 'user' : 'assistant',
                content: msg.content
            }))

        const response = await callBedrock(conversationHistory, selectedModel.value)

        const assistantMessage = {
            id: Date.now() + 1,
            role: 'assistant',
            content: response,
            timestamp: new Date()
        }

        messages.value.push(assistantMessage)
        
    } catch (error) {
        console.error('Error sending message:', error)
        errorMessage.value = error.message || 'Failed to send message'
        
        // Remove the user message if there was an error
        messages.value = messages.value.filter(msg => msg.id !== userMessage.id)
        inputMessage.value = currentInput // Restore the input
    } finally {
        isLoading.value = false
        await nextTick()
        scrollToBottom()
        messageInput.value?.focus()
    }
}

// Clear chat
const clearChat = () => {
    messages.value = []
    errorMessage.value = ''
}

// Scroll to bottom
const scrollToBottom = () => {
    if (messagesContainer.value) {
        messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
    }
}

// Format message (basic HTML support)
const formatMessage = (content) => {
    return content
        .replace(/\n/g, '<br>')
        .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
        .replace(/\*(.*?)\*/g, '<em>$1</em>')
}

// Format time
const formatTime = (date) => {
    return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
}

// Initialize on mount
onMounted(() => {
    initializeAWS()
    messageInput.value?.focus()
})
</script>

<template>
  <div class="chat-container">
    <!-- Header -->
    <ChatHeader 
      :selected-model="selectedModel"
      :is-connected="isConnected"
      @update:selected-model="selectedModel = $event"
    />

    <!-- Messages -->
    <main class="chat-messages" ref="messagesContainer">
      <div v-if="messages.length === 0" class="message assistant">
        <div class="message-avatar">AI</div>
        <div class="message-content">
          <div>Hello! I'm ready to chat with you. This app is set up to connect to AWS Bedrock. How can I help you today?</div>
          <div class="message-time">{{ formatTime(new Date()) }}</div>
        </div>
      </div>

      <ChatMessage
        v-for="message in messages" 
        :key="message.id"
        :message="message"
        :format-message="formatMessage"
        :format-time="formatTime"
      />

      <div v-if="isLoading" class="thinking-indicator">
        <div class="thinking-dots">
          <div class="dot"></div>
          <div class="dot"></div>
          <div class="dot"></div>
        </div>
        <span>AI is thinking...</span>
      </div>

      <div v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </div>
    </main>

    <!-- Input -->
    <ChatInput
      :input-message="inputMessage"
      :is-loading="isLoading"
      @update:input-message="inputMessage = $event"
      @send-message="sendMessage"
      @clear-chat="clearChat"
    />
  </div>
</template>