<script setup lang="ts">
import { Pane, Splitpanes } from 'splitpanes'
import { ref, watchEffect } from 'vue'
import ChatUI from './components/ChatUI.vue'
import JsonEditor from './components/JsonEditor.vue'
import 'splitpanes/dist/splitpanes.css'

// Sample default JSON for demonstration
const defaultJson = JSON.stringify([
  {
    role: 'system',
    content: 'You are a helpful AI assistant.',
  },
  {
    role: 'user',
    content: 'Hello, can you help me with a coding problem?',
  },
  {
    role: 'agent',
    content: 'Of course! I\'d be happy to help with your coding problem. What specifically are you working on?',
  },
], null, 2)

const jsonContent = ref(defaultJson)
const jsonError = ref<string | null>(null)
const parsedMessages = ref([])
const shareUrl = ref('')
const showCopiedMessage = ref(false)

// Load JSON from URL parameter if present
const urlParams = new URLSearchParams(window.location.search)
const jsonParam = urlParams.get('json')

if (jsonParam) {
  try {
    // Decode the base64 parameter
    const decodedJson = atob(jsonParam)
    // Try to parse to validate it's proper JSON
    JSON.parse(decodedJson)
    // If valid, set it as the current content
    jsonContent.value = decodedJson
  }
  catch (e) {
    console.error('Failed to parse JSON from URL parameter:', e)
    jsonError.value = 'Invalid JSON in URL parameter'
  }
}

watchEffect(() => {
  // Update the URL without reloading the page when JSON content changes
  const url = new URL(window.location.href)
  url.searchParams.set('json', btoa(jsonContent.value))
  window.history.replaceState({}, '', url.toString())
})

// Generate shareable URL with base64 encoded JSON
function generateShareUrl() {
  try {
    // Encode current JSON content to base64
    const base64Json = btoa(jsonContent.value)
    // Create URL with parameter
    const url = new URL(window.location.href)
    url.search = `?json=${base64Json}`
    shareUrl.value = url.toString()

    // Copy to clipboard
    navigator.clipboard.writeText(shareUrl.value)
      .then(() => {
        showCopiedMessage.value = true
        setTimeout(() => {
          showCopiedMessage.value = false
        }, 2000)
      })
      .catch((err) => {
        console.error('Failed to copy URL: ', err)
      })
  }
  catch (e) {
    console.error('Error generating share URL:', e)
    jsonError.value = 'Could not generate shareable URL'
  }
}

watchEffect(() => {
  try {
    if (!jsonContent.value.trim()) {
      jsonError.value = 'JSON cannot be empty'
      parsedMessages.value = []
      return
    }

    const parsed = JSON.parse(jsonContent.value)

    if (!Array.isArray(parsed)) {
      jsonError.value = 'JSON must be an array'
      parsedMessages.value = []
      return
    }

    // Define allowed roles for stricter type checking
    const allowedRoles = ['system', 'user', 'agent'] as const
    type ChatRole = typeof allowedRoles[number]

    for (let i = 0; i < parsed.length; i++) {
      const item = parsed[i]

      if (typeof item !== 'object' || item === null) {
        jsonError.value = `Item at index ${i} must be an object`
        parsedMessages.value = []
        return
      }

      if (!('role' in item)) {
        jsonError.value = `Item at index ${i} is missing 'role' property`
        parsedMessages.value = []
        return
      }

      if (!('content' in item)) {
        jsonError.value = `Item at index ${i} is missing 'content' property`
        parsedMessages.value = []
        return
      }

      if (!allowedRoles.includes(item.role as ChatRole)) {
        jsonError.value = `Item at index ${i} has invalid 'role' (must be 'system', 'user', or 'agent')`
        parsedMessages.value = []
        return
      }

      if (typeof item.content !== 'string') {
        jsonError.value = `Item at index ${i} has invalid 'content' (must be a string)`
        parsedMessages.value = []
        return
      }
    }

    jsonError.value = null
    parsedMessages.value = parsed
  }
  catch (e) {
    if (e instanceof Error) {
      jsonError.value = e.message
    }
    else {
      jsonError.value = 'Invalid JSON'
    }
    return []
  }
})
</script>

<template>
  <div class="h-screen w-screen flex flex-col bg-gray-100">
    <header class="bg-gray-800 p-4 text-white">
      <div class="flex items-center justify-between">
        <h1 class="text-2xl font-bold">
          JSON to Chat Converter
        </h1>
        <div class="relative">
          <button
            class="rounded bg-blue-600 px-3 py-1 text-sm text-white hover:bg-blue-700"
            @click="generateShareUrl"
          >
            Share
          </button>
          <div
            v-if="showCopiedMessage"
            class="absolute right-0 mt-2 rounded bg-gray-900 px-2 py-1 text-xs text-white"
          >
            URL copied!
          </div>
        </div>
      </div>
    </header>

    <main class="flex-1 overflow-hidden">
      <Splitpanes class="h-full">
        <Pane
          min-size="30"
          :size="50"
        >
          <div class="h-full flex flex-col">
            <div class="flex items-center justify-between bg-gray-700 px-4 py-2 text-white">
              <div class="font-medium">
                JSON Editor
              </div>
              <div
                v-if="jsonError"
                class="text-sm text-red-300"
              >
                {{ jsonError }}
              </div>
            </div>
            <div class="flex-1 overflow-hidden">
              <JsonEditor v-model="jsonContent" />
            </div>
          </div>
        </Pane>
        <Pane
          min-size="30"
          :size="50"
        >
          <div class="h-full flex flex-col">
            <div class="bg-gray-700 px-4 py-2 text-white font-medium">
              Chat Preview
            </div>
            <div class="flex-1 overflow-hidden bg-white">
              <ChatUI
                :messages="parsedMessages"
                :error="jsonError"
              />
            </div>
          </div>
        </Pane>
      </Splitpanes>
    </main>
  </div>
</template>
