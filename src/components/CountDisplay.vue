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
    <p class="display-4 text-primary mb-3 fw-bold">{{ totalCount }}</p>
    
    <!-- 显示不同来源的数值 -->
    <div class="mb-3 small">
      <span class="badge bg-info me-2">Manual Clicks: {{ manualClickCount }}</span>
      <span class="badge bg-success me-2">Auto Clicks: {{ autoClickCount }}</span>
      <span class="badge bg-warning">A: {{ aCount }}</span>
    </div>
  </div>
</template>