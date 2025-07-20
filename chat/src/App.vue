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

// AWS Configuration using Amplify secrets
const awsConfig = reactive({
    region: import.meta.env.VITE_AWS_REGION || 'us-east-1',
    accessKeyId: import.meta.env.VITE_AWS_ACCESS_KEY_ID || '',
    secretAccessKey: import.meta.env.VITE_AWS_SECRET_ACCESS_KEY || '',
    sessionToken: import.meta.env.VITE_AWS_SESSION_TOKEN || ''
})

let bedrockRuntime = null

// Initialize AWS SDK and test connectivity
const initializeAWS = async () => {
    try {
        console.log('Initializing AWS Bedrock...')
        console.log('AWS Config:', {
            region: awsConfig.region,
            hasAccessKey: !!awsConfig.accessKeyId,
            hasSecretKey: !!awsConfig.secretAccessKey
        })
        
        // Check if we have credentials
        if (!awsConfig.accessKeyId || !awsConfig.secretAccessKey) {
            console.warn('AWS credentials not found in environment variables')
            errorMessage.value = 'AWS credentials not configured'
            return
        }
        
        // Import AWS SDK dynamically
        const { BedrockRuntimeClient, InvokeModelCommand } = await import('@aws-sdk/client-bedrock-runtime')
        
        // Create Bedrock client
        bedrockRuntime = new BedrockRuntimeClient({
            region: awsConfig.region,
            credentials: {
                accessKeyId: awsConfig.accessKeyId,
                secretAccessKey: awsConfig.secretAccessKey,
                ...(awsConfig.sessionToken && { sessionToken: awsConfig.sessionToken })
            }
        })
        
        // Test connectivity with a simple model list operation
        await testConnectivity()
        
        isConnected.value = true
        console.log('AWS Bedrock integration ready')
        
    } catch (error) {
        console.error('Failed to initialize AWS:', error)
        errorMessage.value = `Failed to connect to AWS Bedrock: ${error.message}`
        isConnected.value = false
    }
}

// Test AWS Bedrock connectivity
const testConnectivity = async () => {
    try {
        // Test with a minimal invoke to check permissions
        const testPayload = {
            anthropic_version: "bedrock-2023-05-31",
            max_tokens: 1,
            messages: [{ role: "user", content: "test" }]
        }
        
        const { InvokeModelCommand } = await import('@aws-sdk/client-bedrock-runtime')
        const command = new InvokeModelCommand({
            modelId: selectedModel.value,
            body: JSON.stringify(testPayload),
            contentType: "application/json",
            accept: "application/json"
        })
        
        const response = await bedrockRuntime.send(command)
        console.log('Connectivity test successful')
        return true
        
    } catch (error) {
        console.error('Connectivity test failed:', error)
        throw new Error(`Connectivity test failed: ${error.message}`)
    }
}

// Send message to Bedrock
const callBedrock = async (messages, modelId) => {
    if (!bedrockRuntime || !isConnected.value) {
        throw new Error('AWS Bedrock not connected')
    }
    
    try {
        // Prepare the payload for Claude models
        const payload = {
            anthropic_version: "bedrock-2023-05-31",
            max_tokens: 4000,
            messages: messages,
            temperature: 0.7,
            top_p: 0.9
        }
        
        console.log('Sending request to Bedrock:', { modelId, messageCount: messages.length })
        
        const { InvokeModelCommand } = await import('@aws-sdk/client-bedrock-runtime')
        const command = new InvokeModelCommand({
            modelId: modelId,
            body: JSON.stringify(payload),
            contentType: "application/json",
            accept: "application/json"
        })
        
        const response = await bedrockRuntime.send(command)
        const responseBody = JSON.parse(new TextDecoder().decode(response.body))
        
        console.log('Received response from Bedrock')
        
        // Extract the response content
        if (responseBody.content && responseBody.content.length > 0) {
            return responseBody.content[0].text
        } else {
            throw new Error('Unexpected response format from Bedrock')
        }
        
    } catch (error) {
        console.error('Bedrock API call failed:', error)
        throw new Error(`Bedrock API call failed: ${error.message}`)
    }
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