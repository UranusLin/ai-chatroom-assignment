<template>
  <div class="chat-window-fixed">
    <!-- Floating Trigger Button -->
    <q-btn
      v-if="!isOpen"
      round
      unelevated
      color="accent"
      class="chat-trigger-btn"
      @click="toggleChat"
    >
       <q-icon name="edit" size="20px" />
       <div class="q-ml-sm font-sm text-weight-bold">Ask Nitra AI</div>
    </q-btn>

    <!-- Chat Popup -->
    <div v-if="isOpen" class="chat-popup shadow-10 flex column">
      <!-- Header -->
      <div class="chat-header q-pa-md bg-teal-700 text-white flex column text-left">
        <div class="header-top flex items-center justify-between full-width">
          <div class="header-content flex items-center">
            <div class="q-mr-sm" style="width: 32px; height: auto; display: flex; align-items: center;">
               <img src="~assets/nitra-emblem.svg" alt="Logo" style="width: 100%; height: auto;" />
            </div>
            <div class="text-weight-bold font-md flex items-center" style="line-height: 1.2;">
              Nitra AI
              <img :src="iconMagic" class="q-ml-sm" style="width: 22px; height: auto;" />
            </div>
          </div>
          <q-btn flat round dense icon="close" size="sm" @click="toggleChat" />
        </div>
        <div class="font-xs text-teal-100 q-mt-xs" style="opacity: 0.9;">Hi there, How can we help?</div>
      </div>

      <!-- Messages Area -->
      <div class="messages-container q-pa-md col overflow-auto" ref="scrollArea">
        <ChatMessage
          v-for="(msg, index) in messages"
          :key="index"
          :message="msg"
        />
        <div v-if="isThinking" class="thinking-message">
           <ChatMessage :message="{ role: 'assistant', content: '' }" :isThinking="true" />
        </div>
      </div>

      <!-- Input Area -->
      <ChatInput @send="handleUserMessage" :showActions="messages.length === 1" />
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted, reactive } from 'vue'
import ChatMessage from './ChatMessage.vue'
import ChatInput from './ChatInput.vue'
import { MESSAGE_MOCK_MAP } from 'src/mock/messages.js'

import iconMagic from 'src/assets/icons/icon-magic.svg'

const isOpen = ref(false)
const isThinking = ref(false)
const messages = ref([
  {
    role: 'assistant',
    content: 'Welcome to Nitra AI!',
    timestamp: new Date().toISOString()
  }
])
const scrollArea = ref(null)

const toggleChat = () => {
  isOpen.value = !isOpen.value
}

const scrollToBottom = async () => {
  await nextTick()
  if (scrollArea.value) {
    scrollArea.value.scrollTop = scrollArea.value.scrollHeight
  }
}

const handleUserMessage = async (text) => {
  // Add user message
  messages.value.push({
    role: 'user',
    content: text,
    timestamp: new Date().toISOString()
  })

  await scrollToBottom()

  // Simulate AI Thinking
  isThinking.value = true
  await new Promise(resolve => setTimeout(resolve, 1500))
  isThinking.value = false

  // Find matching mock response
  const trimmedText = text.trim()
  const mockResponse = MESSAGE_MOCK_MAP[trimmedText] ||
                       Object.entries(MESSAGE_MOCK_MAP).find(([key]) => key.toLowerCase() === trimmedText.toLowerCase())?.[1]

  const fullContent = mockResponse ? mockResponse.message.content : "I'm sorry, I don't have a specific answer for that. But I can help you find products or compare vendors!"

  // Split content and suggested question
  let mainContent = fullContent
  let suggestedQuestion = ''

  const splitIndex = fullContent.indexOf('Suggested Question:')
  if (splitIndex !== -1) {
    mainContent = fullContent.substring(0, splitIndex).trim()
    suggestedQuestion = fullContent.substring(splitIndex).trim()
  }

  // Typing effect for main content
  const aiMessage = reactive({
    role: 'assistant',
    content: '',
    timestamp: new Date().toISOString()
  })
  messages.value.push(aiMessage)

  await typeContent(aiMessage, mainContent)

  // Add suggested question as a separate message after typing completes
  if (suggestedQuestion) {
    await new Promise(resolve => setTimeout(resolve, 500))
    messages.value.push({
      role: 'assistant',
      content: suggestedQuestion,
      isSuggested: true,
      timestamp: new Date().toISOString()
    })
    await scrollToBottom()
  }
}

const typeContent = async (messageObj, content) => {
  let currentContent = ''

  // Use a more robust chunking approach to keep markdown blocks atomic
  // This prevents shaking by ensuring blocks like [Image Link](url) appear all at once
  const chunks = []
  let remaining = content

  while (remaining) {
    // Match image link or regular link
    const match = remaining.match(/^(\[.*?\]\(.*?\))/)
    if (match) {
      chunks.push(match[0])
      remaining = remaining.slice(match[0].length)
    } else {
      chunks.push(remaining[0])
      remaining = remaining.slice(1)
    }
  }

  for (let i = 0; i < chunks.length; i++) {
    currentContent += chunks[i]
    messageObj.content = currentContent

    // Only scroll every few chunks to reduce overhead, but always at the end
    if (i % 5 === 0 || i === chunks.length - 1) {
      await scrollToBottom()
    }

    // Typing speed configuration
    const TYPING_DELAY_CHAR = 10
    const TYPING_DELAY_CHUNK = 2

    const delay = chunks[i].length > 1 ? TYPING_DELAY_CHUNK : TYPING_DELAY_CHAR
    await new Promise(resolve => setTimeout(resolve, delay))
  }
}

onMounted(() => {
  // Option to open by default if needed
})
</script>

<style lang="scss" scoped>
.chat-window-fixed {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 9999;
}

.chat-trigger-btn {
  border-radius: 24px;
  padding: 8px 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.chat-popup {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 440px;
  height: 600px;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

.chat-header {
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.messages-container {
  background: #FFFFFF;
  display: flex;
  flex-direction: column;
}

@media (max-width: 500px) {
  .chat-popup {
    width: 90vw;
    height: 80vh;
    right: -12px;
  }
}
</style>
