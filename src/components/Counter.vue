<script setup lang="ts">
import { ref, onUnmounted, computed, onMounted, watch } from 'vue'
import ClickChart from './ClickChart.vue'
import TotalChart from './TotalChart.vue'
import CountDisplay from './CountDisplay.vue'
import ControlButtons from './ControlButtons.vue'
import AutoClicker from './AutoClicker.vue'
import DataManager from './DataManager.vue'

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
const aCount = ref(loadFromStorage('aCount', 0))                    // A的数值

// 计算总数值 - 作为核心资源
const totalCount = computed(() => manualClickCount.value + autoClickCount.value)

// 自动点击相关状态 - 从localStorage加载
const autoClickerActive = ref(loadFromStorage('autoClickerActive', false))
const autoClickerCost = 20
let autoClickInterval: number | null = null

// 手动点击，增加手动点击计数
const increment = () => {
  manualClickCount.value++
}

// 将总数值转换为A
const convertToA = () => {
  if (totalCount.value >= 1) {
    // 优先消耗手动点击数值
    if (manualClickCount.value >= 1) {
      manualClickCount.value--
    } else {
      // 如果手动点击数值不足，则消耗自动点击数值
      autoClickCount.value--
    }
    
    // 增加A的数值
    aCount.value++
  }
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

// 监听数据变化并保存到localStorage
watch(manualClickCount, (newValue) => {
  saveToStorage('manualClickCount', newValue)
})

watch(autoClickCount, (newValue) => {
  saveToStorage('autoClickCount', newValue)
})

watch(aCount, (newValue) => {
  saveToStorage('aCount', newValue)
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
  aCount.value = 0
  autoClickerActive.value = false
  
  // 清除自动点击器
  if (autoClickInterval) {
    clearInterval(autoClickInterval)
    autoClickInterval = null
  }
  
  // 清除localStorage中的所有数据
  localStorage.removeItem('manualClickCount')
  localStorage.removeItem('autoClickCount')
  localStorage.removeItem('aCount')
  localStorage.removeItem('autoClickerActive')
  
  // 清除图表数据
  localStorage.removeItem('clicksPerSecondData')
  localStorage.removeItem('totalPointsData')
  
  // 页面重载，确保所有组件都重新初始化
  reloadPage()
}

// 页面重载方法
const reloadPage = () => {
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
      <!-- 数值显示组件 -->
      <CountDisplay 
        :manual-click-count="manualClickCount"
        :auto-click-count="autoClickCount"
        :a-count="aCount"
      />
      
      <!-- 按钮控制组件 -->
      <ControlButtons
        :total-count="totalCount"
        :auto-clicker-cost="autoClickerCost"
        :auto-clicker-active="autoClickerActive"
        @increment="increment"
        @convert-to-a="convertToA"
        @purchase-auto-clicker="purchaseAutoClicker"
      />
      
      <!-- 自动点击器组件 -->
      <AutoClicker :auto-clicker-active="autoClickerActive" />
      
      <!-- 图表组件 -->
      <div class="d-flex flex-column gap-4 mt-4">
        <div>
          <p class="text-muted small">Clicks Per Second</p>
          <ClickChart :count="totalCount" />
        </div>
        
        <div>
          <p class="text-muted small">Total Points Over Time</p>
          <TotalChart :count="totalCount" />
        </div>
      </div>
      
      <!-- 数据管理组件 -->
      <DataManager
        :manual-click-count="manualClickCount"
        :auto-click-count="autoClickCount"
        :auto-clicker-active="autoClickerActive"
        @import-success="reloadPage"
        @import-error="() => {}"
        @reset="resetGame"
      />
    </div>
  </div>
</template>