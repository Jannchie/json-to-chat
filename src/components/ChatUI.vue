<script setup lang="ts">
import { computed } from 'vue'

interface ChatMessage {
  role: 'system' | 'user' | 'agent'
  content: string
}

defineProps<{
  messages: ChatMessage[]
  error: string | null
}>()

// More subtle role badge styling with less vibrant colors
const roleClasses = computed(() => ({
  system: 'bg-gray-700 text-gray-100 shadow-sm',
  user: 'bg-gray-600 text-gray-100 shadow-sm',
  agent: 'bg-gray-800 text-gray-100 shadow-sm',
}))

// More subtle message bubble styling
const messageClasses = computed(() => ({
  system: 'bg-gray-100 border-gray-200 text-gray-800 shadow-sm',
  user: 'bg-blue-50 border-gray-200 text-gray-800 shadow-sm',
  agent: 'bg-white border-gray-200 text-gray-800 shadow-sm',
}))

// Determine message alignment based on role
const messageAlignment = computed(() => ({
  system: 'items-center',
  user: 'items-end',
  agent: 'items-start',
}))

// Simplified icons for each role
const roleIcons = computed(() => ({
  system: 'i-tabler-settings',
  user: 'i-tabler-user',
  agent: 'i-tabler-robot',
}))
</script>

<template>
  <div class="h-full overflow-auto bg-gray-50 p-4">
    <!-- Empty state with minimal styling -->
    <div
      v-if="messages.length === 0"
      class="h-full flex items-center justify-center"
    >
      <div class="max-w-md border border-gray-200 rounded-md bg-white p-6 text-gray-500">
        <div class="flex flex-col items-center gap-3 text-center">
          <div class="text-gray-600 font-medium">
            No messages to display
          </div>
          <div class="text-sm">
            {{ error || 'Start a conversation!' }}
          </div>
        </div>
      </div>
    </div>

    <!-- Messages with subtle styling -->
    <div
      v-else
      class="mx-auto max-w-4xl py-4 space-y-4"
    >
      <div
        v-for="(message, index) in messages"
        :key="index"
        class="flex flex-col"
        :class="messageAlignment[message.role]"
      >
        <!-- Role badge with subtle styling -->
        <div class="mb-1 flex items-center gap-2">
          <div
            class="flex items-center gap-1 rounded-md px-2 py-1 text-xs font-medium"
            :class="roleClasses[message.role]"
          >
            <i
              class="opacity-70"
              :class="roleIcons[message.role] "
            >{{ roleIcons[message.role] }}</i>
            <span class="capitalize">{{ message.role }}</span>
          </div>
        </div>

        <!-- Message bubble with subtle styling -->
        <div
          class="max-w-[85%] border rounded-md p-3"
          :class="messageClasses[message.role]"
        >
          <div class="whitespace-pre-wrap text-gray-700 leading-relaxed">
            {{ message.content }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Removed animation for a more subtle experience */
</style>
