<template>
  <div class="container">
    <h1>CUBE LUT 到 Hex 纹理转换器</h1>
    <p>请选择一个 <code>.cube</code> 文件:</p>
    <input type="file" ref="fileInput" accept=".cube" @change="handleFileChange" />

    <div class="output-header">
      <h2>输出的十六进制字符串:</h2>
      <button :disabled="!canCopy" @click="copyToClipboard" :class="{ copying: isCopying }">
        {{ copyButtonText }}
      </button>
    </div>
    <pre ref="output">{{ outputText }}</pre>
    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

const fileInput = ref<HTMLInputElement>()
const output = ref<HTMLPreElement>()
const outputText = ref('在这里显示结果...')
const errorMessage = ref('')
const isCopying = ref(false)

const PLACEHOLDER_TEXT = '在这里显示结果...'

const canCopy = computed(() => {
  return outputText.value && outputText.value !== PLACEHOLDER_TEXT && !isCopying.value
})

const copyButtonText = computed(() => {
  return isCopying.value ? '已复制!' : '一键复制'
})

// 核心转换函数
function convertLutToHex(lutContent: string): string {
  const data = lutContent
    .trim()
    .split('\n')
    .filter((line) => {
      const trimmedLine = line.trim()
      return trimmedLine.length > 0 && !trimmedLine.startsWith('#') && !/^[A-Z]/.test(trimmedLine)
    })
    .flatMap((line) => {
      const parts = line.trim().split(/\s+/)
      if (parts.length !== 3 || parts.some((p) => isNaN(parseFloat(p)))) {
        return []
      }
      return parts.map((s) => parseFloat(s)).concat([1.0])
    })

  if (data.length === 0) {
    throw new Error('无效或空的 LUT 文件：未找到有效的数据行。')
  }

  const byteLength = data.length * 2
  const arrayBuffer = new ArrayBuffer(byteLength)
  const dataView = new DataView(arrayBuffer)
  const isLittleEndian = true

  data.forEach((value, index) => {
    const byteOffset = index * 2
    dataView.setFloat16(byteOffset, value, isLittleEndian)
  })

  const uint8Array = new Uint8Array(arrayBuffer)
  const textureHex = Array.from(uint8Array)
    .map((byte) => byte.toString(16).padStart(2, '0'))
    .join('')

  return textureHex
}

// 文件变化处理
function handleFileChange(event: Event) {
  // 重置状态
  outputText.value = PLACEHOLDER_TEXT
  errorMessage.value = ''
  isCopying.value = false

  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (!file) {
    return
  }

  const reader = new FileReader()

  reader.onload = (e) => {
    const fileContent = e.target?.result as string
    try {
      const hexString = convertLutToHex(fileContent)
      outputText.value = hexString
    } catch (error) {
      errorMessage.value = '发生错误: ' + (error as Error).message
      console.error(error)
    }
  }

  reader.onerror = () => {
    errorMessage.value = '读取文件时出错。'
  }

  reader.readAsText(file)
}

// 复制到剪贴板
async function copyToClipboard() {
  const textToCopy = outputText.value

  if (textToCopy && textToCopy !== PLACEHOLDER_TEXT) {
    try {
      await navigator.clipboard.writeText(textToCopy)
      isCopying.value = true
      setTimeout(() => {
        isCopying.value = false
      }, 2000)
    } catch (err) {
      errorMessage.value = '复制失败! 请检查浏览器权限。'
      console.error('Failed to copy text: ', err)
    }
  }
}
</script>

<style scoped>
.container {
  padding: 1.5em;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  line-height: 1.6;
  max-width: 800px;
  margin: 0 auto;
  color: #333;
}

input[type='file'] {
  font-size: 1em;
}

.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 1.5em;
}

h1 {
  margin: 0 0 1em 0;
}

h2 {
  margin: 0;
  font-size: 1.2em;
}

button {
  font-size: 0.9em;
  padding: 0.5em 1em;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f0f0f0;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:hover:not(:disabled) {
  background-color: #e0e0e0;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

button.copying {
  background-color: #4caf50;
  color: white;
  border-color: #4caf50;
}

pre {
  background-color: #f4f4f4;
  padding: 1em;
  border: 1px solid #ddd;
  border-radius: 4px;
  word-wrap: break-word;
  white-space: pre-wrap;
  max-height: 400px;
  overflow-y: auto;
  margin-top: 0.5em;
}

.error {
  color: #d93025;
  font-weight: bold;
  margin-top: 0.5em;
}
</style>
