<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  manualClickCount: number
  autoClickCount: number
  autoClickerActive: boolean
}>()

const emit = defineEmits<{
  (e: 'importSuccess'): void
  (e: 'importError'): void
  (e: 'reset'): void
}>()

const importSuccess = ref(false)
const importError = ref(false)

// 从localStorage加载数据的函数
function loadFromStorage(key: string, defaultValue: any) {
  try {
    const storedValue = localStorage.getItem(key)
    return storedValue ? JSON.parse(storedValue) : defaultValue
  } catch (error) {
    console.error(`Error loading ${key} from localStorage:`, error)
    return defaultValue
  }
}

// 保存数据到localStorage的函数
function saveToStorage(key: string, value: any) {
  try {
    localStorage.setItem(key, JSON.stringify(value))
  } catch (error) {
    console.error(`Error saving ${key} to localStorage:`, error)
  }
}

// 导出数据存档
const exportData = () => {
  // 创建包含所有重要数据的存档对象
  const saveData = {
    version: 1,
    timestamp: new Date().toISOString(),
    data: {
      manualClickCount: props.manualClickCount,
      autoClickCount: props.autoClickCount,
      autoClickerActive: props.autoClickerActive,
      clicksPerSecondData: loadFromStorage('clicksPerSecondData', []),
      totalPointsData: loadFromStorage('totalPointsData', [])
    }
  }
  
  // 转换为JSON字符串
  const saveString = JSON.stringify(saveData)
  
  // 转换为Base64编码，使其更适合作为文本传输
  const saveBase64 = btoa(saveString)
  
  // 创建一个下载链接
  const element = document.createElement('a')
  element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(saveBase64))
  element.setAttribute('download', `clicker-save-${Date.now()}.txt`)
  
  // 模拟点击下载
  element.style.display = 'none'
  document.body.appendChild(element)
  element.click()
  document.body.removeChild(element)
}

// 导入数据存档
const importData = (event: Event) => {
  const fileInput = event.target as HTMLInputElement
  const file = fileInput.files?.[0]
  
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      // 从Base64解码
      const saveBase64 = e.target?.result as string
      const saveString = atob(saveBase64)
      
      // 解析JSON
      const saveData = JSON.parse(saveString)
      
      // 检查版本兼容性
      if (!saveData.version || saveData.version !== 1) {
        throw new Error('Incompatible save version')
      }
      
      // 保存到localStorage
      saveToStorage('manualClickCount', saveData.data.manualClickCount)
      saveToStorage('autoClickCount', saveData.data.autoClickCount)
      saveToStorage('autoClickerActive', saveData.data.autoClickerActive)
      
      // 恢复图表数据
      if (saveData.data.clicksPerSecondData) {
        saveToStorage('clicksPerSecondData', saveData.data.clicksPerSecondData)
      }
      
      if (saveData.data.totalPointsData) {
        saveToStorage('totalPointsData', saveData.data.totalPointsData)
      }
      
      // 显示成功消息
      importSuccess.value = true
      importError.value = false
      emit('importSuccess')
      
      // 3秒后隐藏成功消息
      setTimeout(() => {
        importSuccess.value = false
      }, 2000)
      
      // 清除文件输入
      fileInput.value = ''
    } catch (error) {
      console.error('Error importing save:', error)
      importError.value = true
      importSuccess.value = false
      emit('importError')
      
      // 3秒后隐藏错误消息
      setTimeout(() => {
        importError.value = false
      }, 3000)
      
      // 清除文件输入
      fileInput.value = ''
    }
  }
  
  reader.readAsText(file)
}
</script>

<template>
  <div class="mt-4">
    <div class="alert alert-success" v-if="importSuccess">
      Save data imported successfully!
    </div>
    <div class="alert alert-danger" v-if="importError">
      Error importing save data. Please check file format.
    </div>
    
    <div class="btn-group">
      <button @click="exportData" class="btn btn-outline-primary btn-sm">
        Export Save
      </button>
      <label class="btn btn-outline-primary btn-sm">
        Import Save
        <input type="file" accept=".txt" @change="importData" class="d-none" />
      </label>
      <button @click="emit('reset')" class="btn btn-outline-danger btn-sm">
        Reset
      </button>
    </div>
  </div>
</template> 