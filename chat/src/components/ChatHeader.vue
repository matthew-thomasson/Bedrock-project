<script setup>
defineProps({
  selectedModel: String,
  isConnected: Boolean
})

defineEmits(['update:selectedModel'])
</script>

<template>
  <header class="chat-header">
    <h1>🤖 Bedrock Chat</h1>
    <div class="model-selector">
      <label for="model-select" style="font-size: 0.75rem;">Model:</label>
      <select 
        id="model-select" 
        class="model-select" 
        :value="selectedModel"
        @change="$emit('update:selectedModel', $event.target.value)"
      >
        <option value="anthropic.claude-3-5-sonnet-20241022-v2:0">Claude 3.5 Sonnet</option>
        <option value="anthropic.claude-3-haiku-20240307-v1:0">Claude 3 Haiku</option>
        <option value="amazon.titan-text-premier-v1:0">Titan Text Premier</option>
      </select>
    </div>
    <div class="connection-status">
      <div :class="['status-indicator', { disconnected: !isConnected }]"></div>
      <span>{{ isConnected ? 'Connected' : 'Disconnected' }}</span>
    </div>
  </header>
</template>