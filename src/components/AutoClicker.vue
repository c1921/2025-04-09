<script setup lang="ts">
const props = defineProps<{
  autoClickerActive: boolean
}>()
</script>

<template>
  <div class="upgrades-container mt-4">
    <div class="card upgrade-card">
      <div class="card-body">
        <p class="upgrade-title">Auto Clicker</p>
        <p class="text-muted small">Adds 1 point per second automatically</p>
        <div class="auto-clicker-status">
          <span v-if="autoClickerActive" class="badge bg-success">Activated</span>
          <span v-else class="badge bg-secondary">Not Active</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
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
</style> 