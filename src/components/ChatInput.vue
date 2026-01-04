<template>
  <div class="chat-input-container q-pa-sm bg-white">
    <!-- Carousel Action Buttons -->
    <div class="action-carousel q-mb-sm" v-if="showActions">
      <transition name="slide-fade" mode="out-in">
        <div :key="currentActionIndex" class="action-item flex items-center">
          <img :src="carouselItems[currentActionIndex].icon" alt="" class="action-icon q-mr-sm" />
          <span class="font-sm text-grey-8">{{ carouselItems[currentActionIndex].text }}</span>
        </div>
      </transition>
    </div>

    <div class="input-row flex items-center q-pt-sm">
      <q-input
        ref="inputRef"
        v-model="text"
        placeholder="Say something..."
        borderless
        dense
        autogrow
        class="col"
        @keydown.enter.prevent.exact="handleSend"
      >
        <template v-slot:append>
          <img :src="iconAttach" alt="Attach" class="attach-icon cursor-pointer" />
        </template>
      </q-input>

      <img
        :src="iconSend"
        alt="Send"
        class="send-icon q-ml-sm cursor-pointer"
        :class="{ 'send-disabled': !text.trim() }"
        @click="handleSend"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue'

import iconList from 'src/assets/icons/icon-list.svg'
import iconCart from 'src/assets/icons/icon-cart.svg'
import iconPrice from 'src/assets/icons/icon-price.svg'
import iconSearch from 'src/assets/icons/icon-search.svg'
import iconThumbup from 'src/assets/icons/icon-thumbup.svg'
import iconSend from 'src/assets/icons/icon-send.svg'
import iconAttach from 'src/assets/icons/icon-attach.svg'

const carouselItems = [
  { icon: iconList, text: 'Upload your supplier list' },
  { icon: iconCart, text: 'Check if Avastin is in stock' },
  { icon: iconPrice, text: "Check if there's a better price for Xeomin" },
  { icon: iconSearch, text: 'What are some generic options for Restylane' },
  { icon: iconThumbup, text: "What's the best product for Xeomin" }
]

const props = defineProps({
  showActions: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['send'])
const text = ref('')
const inputRef = ref(null)

const currentActionIndex = ref(0)
let intervalId = null

const handleSend = () => {
  if (text.value.trim()) {
    emit('send', text.value)
    text.value = ''

    // Reset the autogrow height after clearing text
    nextTick(() => {
      if (inputRef.value && inputRef.value.$el) {
        const textarea = inputRef.value.$el.querySelector('textarea')
        if (textarea) {
          textarea.style.height = 'auto'
        }
      }
    })
  }
}

onMounted(() => {
  intervalId = setInterval(() => {
    currentActionIndex.value = (currentActionIndex.value + 1) % carouselItems.length
  }, 3000)
})

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId)
  }
})
</script>

<style lang="scss" scoped>
.chat-input-container {
  width: 100%;
  box-sizing: border-box;
}

.action-carousel {
  height: 32px;
  overflow: hidden;
  position: relative;
}

.action-item {
  padding: 4px 8px;
}

.action-icon {
  width: 16px;
  height: 16px;
}

.input-row {
  background: white;
  border-radius: 4px;
  border-top: 1px solid $gray-100;
}

.attach-icon {
  width: 20px;
  height: 20px;
  opacity: 0.7;
  transition: opacity 0.2s;

  &:hover {
    opacity: 1;
  }
}

.send-icon {
  width: 36px;
  height: 36px;
  transition: opacity 0.2s;

  &.send-disabled {
    opacity: 0.4;
    pointer-events: none;
  }
}

/* Slide fade animation from bottom to top */
.slide-fade-enter-active {
  transition: all 0.4s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.3s ease-in;
}

.slide-fade-enter-from {
  transform: translateY(20px);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}
</style>
