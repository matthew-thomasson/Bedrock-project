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

// AWS Configuration using Amplify environment variables
const awsConfig = reactive({
    region: import.meta.env.VITE_AWS_REGION || 'us-east-1',
    accessKeyId: import.meta.env.VITE_AWS_ACCESS_KEY_ID || '',
    secretAccessKey: import.meta.env.VITE_AWS_SECRET_ACCESS_KEY || '',
    sessionToken: import.meta.env.VITE_AWS_SESSION_TOKEN || ''
})

let bedrockRuntime = null

// Initialize AWS SDK (stubbed)
const initializeAWS = () => {
    try {
        // This will be uncommented when AWS credentials are available
        /*
        AWS.config.update({
            region: awsConfig.region,
            accessKeyId: awsConfig.accessKeyId,
            secretAccessKey: awsConfig.secretAccessKey,
            sessionToken: awsConfig.sessionToken
        });
        
        bedrockRuntime = new AWS.BedrockRuntime({
            region: awsConfig.region
        });
        */
        
        // Simulate connection for demo
        setTimeout(() => {
            isConnected.value = true
        }, 1000)
        
        console.log('AWS Bedrock integration ready (stubbed)')
    } catch (error) {
        console.error('Failed to initialize AWS:', error)
        errorMessage.value = 'Failed to connect to AWS Bedrock'
    }
}

// Send message to Bedrock (stubbed)
const callBedrock = async (messages, modelId) => {
    // This is where the actual Bedrock API call will be made
    // For now, return a stubbed response
    
    return new Promise((resolve) => {
        setTimeout(() => {
            const responses = [
                "I'm a stubbed response from AWS Bedrock. When you integrate with the actual service, I'll provide real AI-generated responses!",
                "This is a simulated response. The actual Bedrock integration will use the model: " + modelId,
                "Hello! I'm ready to help once you connect to AWS Bedrock. This is just a placeholder response for now.",
                "Great question! Once connected to Bedrock, I'll be able to provide much more detailed and helpful responses.",
                "I'm running in demo mode. Connect to AWS Bedrock to unlock my full capabilities!"
            ]
            
            const randomResponse = responses[Math.floor(Math.random() * responses.length)]
            resolve(randomResponse)
        }, 1500 + Math.random() * 1000) // Random delay between 1.5-2.5s
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