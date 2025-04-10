<script setup lang="ts">
import { ref, onUnmounted, computed, onMounted, watch } from 'vue'
import ClickChart from './ClickChart.vue'
import TotalChart from './TotalChart.vue'

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

// 区分不同来源的数值 - 从localStorage加载初始值
const manualClickCount = ref(loadFromStorage('manualClickCount', 0))  // 手动点击产生的数值
const autoClickCount = ref(loadFromStorage('autoClickCount', 0))    // 自动点击产生的数值

// 计算总数值 - 作为核心资源
const totalCount = computed(() => manualClickCount.value + autoClickCount.value)

// 自动点击相关状态 - 从localStorage加载
const autoClickerActive = ref(loadFromStorage('autoClickerActive', false))
const autoClickerCost = 20
let autoClickInterval: number | null = null

// 导入导出状态
const importSuccess = ref(false)
const importError = ref(false)

// 手动点击，增加手动点击计数
const increment = () => {
  manualClickCount.value++
}

// 购买自动点击器
const purchaseAutoClicker = () => {
  // 使用总数值作为可花费资源
  if (totalCount.value >= autoClickerCost && !autoClickerActive.value) {
    // 减少手动点击数值，优先消耗这部分
    if (manualClickCount.value >= autoClickerCost) {
      manualClickCount.value -= autoClickerCost
    } else {
      // 如果手动点击数值不足，则消耗一部分自动点击数值
      const remainingCost = autoClickerCost - manualClickCount.value
      manualClickCount.value = 0
      autoClickCount.value -= remainingCost
    }
    
    activateAutoClicker()
  }
}

// 激活自动点击器
const activateAutoClicker = () => {
  autoClickerActive.value = true
  
  // 每秒自动增加1点，记录到自动点击数值中
  autoClickInterval = setInterval(() => {
    autoClickCount.value++
  }, 1000) as unknown as number
}

// 导出数据存档
const exportData = () => {
  // 创建包含所有重要数据的存档对象
  const saveData = {
    version: 1,
    timestamp: new Date().toISOString(),
    data: {
      manualClickCount: manualClickCount.value,
      autoClickCount: autoClickCount.value,
      autoClickerActive: autoClickerActive.value,
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
      
      // 恢复数据
      manualClickCount.value = saveData.data.manualClickCount
      autoClickCount.value = saveData.data.autoClickCount
      autoClickerActive.value = saveData.data.autoClickerActive
      
      // 保存到localStorage
      saveToStorage('manualClickCount', manualClickCount.value)
      saveToStorage('autoClickCount', autoClickCount.value)
      saveToStorage('autoClickerActive', autoClickerActive.value)
      
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
      
      // 3秒后隐藏成功消息
      setTimeout(() => {
        importSuccess.value = false
        // 刷新页面加载新数据
        window.location.reload()
      }, 2000)
      
      // 清除文件输入
      fileInput.value = ''
    } catch (error) {
      console.error('Error importing save:', error)
      importError.value = true
      importSuccess.value = false
      
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

// 监听数据变化并保存到localStorage
watch(manualClickCount, (newValue) => {
  saveToStorage('manualClickCount', newValue)
})

watch(autoClickCount, (newValue) => {
  saveToStorage('autoClickCount', newValue)
})

watch(autoClickerActive, (newValue) => {
  saveToStorage('autoClickerActive', newValue)
})

// 组件挂载时检查是否需要激活自动点击器
onMounted(() => {
  if (autoClickerActive.value) {
    // 如果自动点击器应该是激活状态，重新启动它
    autoClickInterval = setInterval(() => {
      autoClickCount.value++
    }, 1000) as unknown as number
  }
})

// 重置游戏数据
const resetGame = () => {
  // 清除所有计数和状态
  manualClickCount.value = 0
  autoClickCount.value = 0
  autoClickerActive.value = false
  
  // 清除自动点击器
  if (autoClickInterval) {
    clearInterval(autoClickInterval)
    autoClickInterval = null
  }
  
  // 清除localStorage中的所有数据
  localStorage.removeItem('manualClickCount')
  localStorage.removeItem('autoClickCount')
  localStorage.removeItem('autoClickerActive')
  
  // 清除图表数据
  localStorage.removeItem('clicksPerSecondData')
  localStorage.removeItem('totalPointsData')
  
  // 页面重载，确保所有组件都重新初始化
  window.location.reload()
}

// 组件卸载时清除自动点击定时器
onUnmounted(() => {
  if (autoClickInterval) {
    clearInterval(autoClickInterval)
  }
})
</script>

<template>
  <div class="card shadow-sm p-4 mb-4">
    <div class="card-body text-center">
      <!-- 显示核心总数值 -->
      <p class="display-4 text-primary mb-3">{{ totalCount }}</p>
      
      <!-- 显示不同来源的数值 -->
      <div class="counts-breakdown mb-3">
        <span class="badge bg-info me-2">Manual Clicks: {{ manualClickCount }}</span>
        <span class="badge bg-success">Auto Clicks: {{ autoClickCount }}</span>
      </div>
      
      <button @click="increment" class="btn btn-primary">Click</button>
      
      <div class="upgrades-container mt-4">
        <div class="card upgrade-card">
          <div class="card-body">
            <p class="upgrade-title">Auto Clicker</p>
            <p class="text-muted small">Adds 1 point per second automatically</p>
            <button 
              @click="purchaseAutoClicker" 
              class="btn" 
              :class="{'btn-success': totalCount >= autoClickerCost && !autoClickerActive, 'btn-secondary': totalCount < autoClickerCost || autoClickerActive}" 
              :disabled="totalCount < autoClickerCost || autoClickerActive">
              <span v-if="!autoClickerActive">Unlock (Cost: {{ autoClickerCost }} points)</span>
              <span v-else>Activated</span>
            </button>
          </div>
        </div>
      </div>
      
      <div class="charts-container">
        <div class="mt-4">
          <p class="text-muted small">Clicks Per Second</p>
          <ClickChart :count="totalCount" />
        </div>
        
        <div class="mt-4">
          <p class="text-muted small">Total Points Over Time</p>
          <TotalChart :count="totalCount" />
        </div>
      </div>
      
      <!-- 数据导入导出 -->
      <div class="mt-4 save-controls">
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
          <button @click="resetGame" class="btn btn-outline-danger btn-sm">
            Reset
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 我们可以添加一些自定义样式来补充Bootstrap */
.card {
  max-width: 800px;
  margin: 0 auto;
  border-radius: 10px;
}

.display-4 {
  font-weight: bold;
}

.counts-breakdown {
  font-size: 0.9rem;
}

.charts-container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.upgrades-container {
  display: flex;
  justify-content: center;
  margin-top: 1rem;
}

.upgrade-card {
  width: 300px;
  margin: 0 0.5rem;
  border: 1px solid #dee2e6;
  transition: all 0.3s ease;
}

.upgrade-title {
  font-weight: bold;
  margin-bottom: 0.25rem;
}

.upgrade-card:hover {
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.save-controls {
  margin-top: 1.5rem;
}

.save-controls .alert {
  margin-bottom: 1rem;
}

@media (min-width: 992px) {
  .card {
    max-width: 1200px;
  }
}
</style>