<script setup lang="ts">
defineProps<{
  totalCount: number
  autoClickerCost: number
  autoClickerActive: boolean
}>()

const emit = defineEmits<{
  (e: 'increment'): void
  (e: 'convertToA'): void
  (e: 'purchaseAutoClicker'): void
}>()
</script>

<template>
  <div class="d-flex justify-content-center gap-2 mb-3">
    <button @click="emit('increment')" class="btn btn-primary">Click</button>
    <button 
      @click="emit('convertToA')" 
      class="btn btn-warning"
      :disabled="totalCount < 1">
      Convert to A (Cost: 1)
    </button>
    <button 
      @click="emit('purchaseAutoClicker')" 
      class="btn btn-success"
      :disabled="totalCount < autoClickerCost || autoClickerActive">
      <span v-if="!autoClickerActive">Buy Auto Clicker ({{ autoClickerCost }})</span>
      <span v-else>Auto Active</span>
    </button>
  </div>
</template> 