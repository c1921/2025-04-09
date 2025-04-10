<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  manualClickCount: number
  autoClickCount: number
  aCount: number
}>()

const totalCount = computed(() => props.manualClickCount + props.autoClickCount)
</script>

<template>
  <div class="text-center">
    <!-- 显示核心总数值 -->
    <p class="display-4 text-primary mb-3">{{ totalCount }}</p>
    
    <!-- 显示不同来源的数值 -->
    <div class="counts-breakdown mb-3">
      <span class="badge bg-info me-2">Manual Clicks: {{ manualClickCount }}</span>
      <span class="badge bg-success me-2">Auto Clicks: {{ autoClickCount }}</span>
      <span class="badge bg-warning">A: {{ aCount }}</span>
    </div>
  </div>
</template>

<style scoped>
.display-4 {
  font-weight: bold;
}

.counts-breakdown {
  font-size: 0.9rem;
}
</style> 