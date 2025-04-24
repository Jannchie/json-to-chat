<script setup lang="ts">
import loader from '@monaco-editor/loader'
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'

// Define props and emits
const props = defineProps<{
  modelValue: string
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()

// Create refs and state variables
const editorContainer = ref<HTMLElement | null>(null)
let editor: any = null
let monaco: any = null
let isUpdatingFromModelValue = false
let lastEmittedValue = props.modelValue

// Editor initialization function
async function initializeEditor() {
  // Load Monaco dynamically
  monaco = await loader.init()

  // Configure Monaco only if it was successfully loaded
  if (monaco && editorContainer.value) {
    // Create editor instance
    editor = monaco.editor.create(editorContainer.value, {
      value: props.modelValue,
      language: 'json',
      theme: 'vs-dark',
      automaticLayout: true,
      minimap: { enabled: false },
      scrollBeyondLastLine: false,
      lineNumbers: 'on',
      roundedSelection: true,
      fontSize: 14,
      formatOnPaste: true,
      tabSize: 2,
      wordWrap: 'on',
    })

    // Handle content changes
    editor.onDidChangeModelContent(() => {
      // Only emit update when not being updated from props
      if (!isUpdatingFromModelValue) {
        const value = editor.getValue()
        // Avoid unnecessary updates
        if (value !== lastEmittedValue) {
          lastEmittedValue = value
          emit('update:modelValue', value)
        }
      }
    })

    // Update initial content
    if (props.modelValue && props.modelValue !== editor.getValue()) {
      editor.setValue(props.modelValue)
    }
  }
}

// Watch for external changes to modelValue
watch(() => props.modelValue, (newValue) => {
  // Avoid updating if the editor isn't initialized yet
  if (!editor) {
    return
  }

  // Skip if the value is already correct in the editor
  if (newValue === editor.getValue()) {
    return
  }

  // Prevent infinite loop by setting a flag
  isUpdatingFromModelValue = true
  try {
    editor.setValue(newValue)
    lastEmittedValue = newValue
  }
  finally {
    // Ensure flag is reset even if an error occurs
    setTimeout(() => {
      isUpdatingFromModelValue = false
    }, 0)
  }
}, { immediate: false })

// Initialize editor
onMounted(() => {
  initializeEditor()
})

// Clean up resources
onBeforeUnmount(() => {
  if (editor) {
    editor.dispose()
    editor = null
  }
})
</script>

<template>
  <div
    ref="editorContainer"
    class="h-full w-full border border-gray-700"
  />
</template>
