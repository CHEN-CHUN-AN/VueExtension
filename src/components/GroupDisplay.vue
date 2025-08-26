<script setup>
defineProps({
  groups: { type: Array, required: true },
});
</script>

<template>
  <div v-if="groups.length > 0" class="groups-section">
    <h3 class="groups-title">分組結果</h3>
    <div class="groups-container">
      <div v-for="(group, index) in groups" :key="index" class="group-card">
        <div class="group-header">
          <h4 class="group-name">第 {{ index + 1 }} 組</h4>
          <span class="group-count">{{ group.length }} 人</span>
        </div>
        <ul class="group-members">
          <li v-for="member in group" :key="member" class="group-member">
            {{ member }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
.groups-section {
  margin-top: 1.5rem;
}

.groups-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--color-heading);
  margin-bottom: 1rem;
}

.groups-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 1rem;
}

.group-card {
  background: var(--c-white);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  transition: all 0.2s ease-in-out;
}

.group-card:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-md);
}

.group-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid var(--color-border);
}

.group-name {
  font-size: 1rem;
  font-weight: 600;
  color: var(--color-primary);
}

.group-count {
  font-size: 0.8rem;
  font-weight: 500;
  color: var(--color-text-muted);
  background-color: var(--color-background-soft);
  padding: 0.2rem 0.5rem;
  border-radius: var(--radius-full);
}

.group-members {
  list-style: none;
  padding: 0.5rem 1rem 1rem;
  margin: 0;
}

.group-member {
  padding: 0.5rem 0;
  font-size: 0.9rem;
  color: var(--color-text);
  border-bottom: 1px solid var(--color-border);
}

.group-member:last-child {
  border-bottom: none;
  padding-bottom: 0;
}
</style>
