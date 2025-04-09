<script setup lang="ts">
import { ref, onUnmounted, computed } from 'vue'
import ClickChart from './ClickChart.vue'
import TotalChart from './TotalChart.vue'

// 区分不同来源的数值
const manualClickCount = ref(0)  // 手动点击产生的数值
const autoClickCount = ref(0)    // 自动点击产生的数值

// 计算总数值 - 作为核心资源
const totalCount = computed(() => manualClickCount.value + autoClickCount.value)

// 自动点击相关状态
const autoClickerActive = ref(false)
const autoClickerCost = 20
let autoClickInterval: number | null = null

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

@media (min-width: 992px) {
  .card {
    max-width: 1200px;
  }
}
</style>