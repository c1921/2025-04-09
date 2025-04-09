<script setup lang="ts">
import { ref } from 'vue'
import ClickChart from './ClickChart.vue'
import TotalChart from './TotalChart.vue'

const count = ref(0)

const increment = () => {
  count.value++
}
</script>

<template>
  <div class="card shadow-sm p-4 mb-4">
    <div class="card-body text-center">
      <p class="display-4 text-primary mb-4">{{ count }}</p>
      <button @click="increment" class="btn btn-primary">点击</button>
      
      <div class="charts-container">
        <div class="mt-4">
          <p class="text-muted small">每秒点击次数</p>
          <ClickChart :count="count" />
        </div>
        
        <div class="mt-4">
          <p class="text-muted small">点击总数变化</p>
          <TotalChart :count="count" />
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

.charts-container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

@media (min-width: 992px) {
  .card {
    max-width: 1200px;
  }
}
</style>