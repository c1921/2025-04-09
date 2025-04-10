<script setup lang="ts">
defineProps<{
  units: Array<{
    id: number
    alpha: number
    beta: number
  }>
}>()
</script>

<template>
  <div class="mt-4">
    <h5 class="text-center mb-3">Units</h5>
    <div class="d-flex flex-wrap justify-content-center gap-3">
      <div v-for="unit in units" :key="unit.id" class="card shadow-sm" style="width: 200px;">
        <div class="card-body">
          <div class="d-flex justify-content-between mb-2">
            <span class="badge bg-primary">α: {{ unit.alpha }}</span>
            <span class="badge bg-info">β: {{ unit.beta }}</span>
          </div>
          <div class="text-center">
            <span class="badge bg-secondary">ID: {{ unit.id }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template> 