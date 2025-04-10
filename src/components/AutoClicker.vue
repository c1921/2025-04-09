<script setup lang="ts">
defineProps<{
  autoClickerActive: boolean
}>()
</script>

<template>
  <div class="d-flex justify-content-center mt-4">
    <div class="card shadow-sm" style="width: 300px;">
      <div class="card-body">
        <p class="fw-bold mb-1">Auto Clicker</p>
        <p class="text-muted small">Adds 1 point per second automatically</p>
        <div>
          <span v-if="autoClickerActive" class="badge bg-success">Activated</span>
          <span v-else class="badge bg-secondary">Not Active</span>
        </div>
      </div>
    </div>
  </div>
</template>
