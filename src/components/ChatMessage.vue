<template>
  <div
    class="chat-message-container q-mb-md flex"
    :class="isAi ? 'ai-message' : 'user-message justify-end'"
  >
    <div v-if="isAi" class="avatar-container q-mr-sm flex items-start">
      <q-avatar size="32px">
        <img src="~assets/icons/ai-avatar.svg" alt="AI" />
      </q-avatar>
    </div>

    <div
      class="message-bubble q-pa-md"
      :class="[
        isAi ? 'ai-bubble' : 'user-bubble',
        message.isSuggested ? 'suggested-bubble' : ''
      ]"
    >
      <div class="message-content font-sm" v-html="formattedContent"></div>

      <!-- Rendering thinking state -->
      <div v-if="isThinking && isAi" class="thinking-dots flex q-gutter-x-xs q-mt-xs">
        <div class="dot"></div>
        <div class="dot"></div>
        <div class="dot"></div>
      </div>
    </div>

    <div v-if="!isAi" class="message-timestamp font-xss text-grey-4 q-mt-xs text-right full-width">
       {{ formattedTime }}
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  message: {
    type: Object,
    required: true
  },
  isThinking: {
    type: Boolean,
    default: false
  }
})

const isAi = computed(() => props.message.role === 'assistant')

const formattedContent = computed(() => {
  if (props.isThinking) return 'Thinking...'

  let content = props.message.content || ''

  // Remove "Suggested Question: " prefix if it exists in the content for separate bubble
  if (props.message.isSuggested) {
    content = content.replace(/^Suggested Question: /g, '')
  }

  // Handle bold: **text**
  content = content.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')

  // Handle image links: [Image Link](url)
  // Use a fixed container to prevent layout shifts during typing
  content = content.replace(
    /\[Image Link\]\((https?:\/\/[^\s)]+)\)/g,
    '<div class="product-image-container"><img src="$1" class="product-image" onerror="this.src=\'https://placehold.co/100x100?text=No+Image\'; this.onerror=null;" /></div>'
  )

  // Handle other links: [text](url)
  content = content.replace(/\[(.*?)\]\((.*?)\)/g, '<a href="$2" target="_blank" class="chat-link">$1</a>')

  // Handle new lines
  content = content.replace(/\n/g, '<br>')

  return content
})

const formattedTime = computed(() => {
  if (!props.message.timestamp) return ''
  const date = new Date(props.message.timestamp)
  return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
})
</script>

<style lang="scss" scoped>
.chat-message-container {
  max-width: 100%;
}

.message-bubble {
  border-radius: 12px;
  max-width: 85%;
  word-wrap: break-word;
}

.ai-bubble {
  background-color: #F2F4F7;
  color: #1D2939;
  border-bottom-left-radius: 2px;
}

.user-bubble {
  background-color: $teal-50;
  color: $teal-700;
  border-bottom-right-radius: 2px;
}

.suggested-bubble {
  background-color: #F2F4F7;
  color: #1D2939;
  border-bottom-left-radius: 2px;
  margin-top: 8px;
}

.message-content {
  line-height: 1.5;

  :deep(strong) {
    font-weight: 600;
  }

  :deep(.chat-link) {
    color: $teal-700;
    text-decoration: underline;
    font-weight: 500;
  }

  :deep(.product-image-container) {
    width: 100px;
    height: 100px;
    margin: 8px 0;
    background-color: #F2F4F7;
    border-radius: 4px;
    display: block;
    overflow: hidden;
  }

  :deep(.product-image) {
    width: 100px;
    height: 100px;
    object-fit: contain;
    display: block;
  }
}

.thinking-dots {
  .dot {
    width: 6px;
    height: 6px;
    background-color: #ABB5BA;
    border-radius: 50%;
    animation: bounce 1.4s infinite ease-in-out both;

    &:nth-child(1) { animation-delay: -0.32s; }
    &:nth-child(2) { animation-delay: -0.16s; }
  }
}

@keyframes bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1.0); }
}
</style>
