<script setup>
defineProps({
  message: Object,
  formatMessage: Function,
  formatTime: Function
})
</script>

<template>
  <div :class="['message', message.role]">
    <div class="message-avatar">{{ message.role === 'user' ? 'You' : 'AI' }}</div>
    <div class="message-content">
      <div v-html="formatMessage(message.content)"></div>
      <div class="message-time">{{ formatTime(message.timestamp) }}</div>
    </div>
  </div>
</template>