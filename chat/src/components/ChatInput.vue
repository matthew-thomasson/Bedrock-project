<script setup>
defineProps({
  inputMessage: String,
  isLoading: Boolean
})

const emit = defineEmits(['update:inputMessage', 'send-message', 'clear-chat'])

const handleEnterKey = (event) => {
  if (event.shiftKey) {
    // Allow Shift+Enter for new line
    return
  } else {
    // Send message on Enter
    event.preventDefault()
    emit('send-message')
  }
}
</script>

<template>
  <footer class="chat-input-container">
    <div class="chat-input-form">
      <div class="input-wrapper">
        <textarea
          :value="inputMessage"
          @input="$emit('update:inputMessage', $event.target.value)"
          @keydown.enter="handleEnterKey"
          placeholder="Type your message... (Enter to send, Shift+Enter for new line)"
          class="chat-input"
          :disabled="isLoading"
        ></textarea>
        <div class="input-actions">
          <button 
            @click="$emit('send-message')"
            :disabled="!inputMessage.trim() || isLoading"
            class="send-button"
            title="Send message"
          >
            <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
              <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
            </svg>
          </button>
        </div>
      </div>
      <button @click="$emit('clear-chat')" class="clear-button">Clear Chat</button>
    </div>
  </footer>
</template>