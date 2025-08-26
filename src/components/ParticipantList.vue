<script setup>


defineProps({
  currentList: { type: Array, required: true },
  isListVisible: { type: Boolean, required: true },
  groups: { type: Array, required: true },
  groupCount: { type: Number, required: true },
});

const emit = defineEmits([
  'toggle-list-visibility',
  'clear-list',
  'remove-item',
  'update:groupCount',
  'group-participants',
]);
</script>

<template>
  <div class="participant-list-section">
    <!-- List Header -->
    <div class="list-header">
      <h3 class="list-title">
        📋 當前名單 ({{ currentList.length }} 人)
      </h3>
      <div class="list-header-buttons">
        <button @click="emit('toggle-list-visibility')" class="btn btn-sm btn-secondary">
          {{ isListVisible ? '🔼 收起' : '🔽 展開' }}
        </button>
        <button
          @click="emit('clear-list')"
          :disabled="currentList.length === 0"
          class="btn btn-sm btn-danger"
        >
          🧹 清空
        </button>
      </div>
    </div>

    <!-- Collapsible List -->
    <Transition name="list-collapse">
      <div v-show="isListVisible" class="list-items-wrapper">
        <div v-if="currentList.length > 0" class="list-items">
            <div
              v-for="(item, index) in currentList"
              :key="index"
              class="list-item"
            >
              <span class="item-content">{{ item }}</span>
              <button @click="emit('remove-item', item)" class="remove-button">
                &times;
              </button>
            </div>
        </div>
         <div v-else class="empty-list-message">
          <p>名單是空的，請在上方設定中新增。</p>
        </div>
      </div>
    </Transition>

    <!-- Grouping Section -->
    <div class="group-section">
        <h3 class="list-title">👥 名單分組</h3>
        <div class="group-controls">
          <label for="group-count-input" class="group-label">分成</label>
          <input
            id="group-count-input"
            type="number"
            :value="groupCount"
            @input="emit('update:groupCount', $event.target.value)"
            min="2"
            class="group-count-input"
          />
          <label for="group-count-input" class="group-label">組</label>
          <button @click="emit('group-participants')" class="btn btn-primary">執行分組</button>
        </div>
    </div>
  </div>
</template>

<style scoped>
.participant-list-section {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
}

/* List Header */
.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.list-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--color-heading);
  margin: 0;
}

.list-header-buttons {
  display: flex;
  gap: 0.5rem;
}

/* List Items */
.list-items-wrapper {
  overflow: hidden;
}

.list-items {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 0.5rem;
  max-height: 250px;
  overflow-y: auto;
  padding: 0.25rem;
  margin: -0.25rem; /* Offset padding for scrollbar */
}

.list-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--color-background-soft);
  padding: 0.5rem 0.75rem;
  border-radius: var(--radius-md);
  border: 1px solid var(--color-border);
  font-size: 0.9rem;
  transition: all 0.2s ease-in-out;
}

.item-content {
  font-weight: 500;
  color: var(--color-text);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.remove-button {
  background: transparent;
  border: none;
  cursor: pointer;
  color: var(--color-text-muted);
  font-size: 1.25rem;
  line-height: 1;
  padding: 0 0.25rem;
  border-radius: var(--radius-sm);
}

.remove-button:hover {
  color: var(--c-red-600);
  background-color: var(--c-red-100);
}

.empty-list-message {
    text-align: center;
    padding: 2rem;
    background-color: var(--color-background-soft);
    border-radius: var(--radius-md);
    color: var(--color-text-muted);
}

/* Grouping Section */
.group-section {
    margin-top: 1rem;
    padding-top: 1.5rem;
    border-top: 1px solid var(--color-border);
}

.group-controls {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.group-label {
    font-size: 0.9rem;
    font-weight: 600;
    color: var(--color-heading);
}

.group-count-input {
  padding: 0.5rem 0.75rem;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: 1rem;
  background: var(--c-white);
  transition: border-color 0.2s, box-shadow 0.2s;
  width: 70px;
  text-align: center;
}

.group-count-input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--c-blue-100);
}

/* Buttons */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.75rem 1.5rem;
  border: 1px solid transparent;
  border-radius: var(--radius-md);
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  user-select: none;
}

.btn-sm {
    padding: 0.4rem 0.8rem;
    font-size: 0.8rem;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-primary {
  background-color: var(--color-primary);
  color: var(--c-white);
  border-color: var(--color-primary);
}

.btn-primary:not(:disabled):hover {
  background-color: var(--color-primary-hover);
  border-color: var(--color-primary-hover);
}

.btn-secondary {
  background-color: var(--color-background-soft);
  color: var(--color-text);
  border-color: var(--color-border);
}

.btn-secondary:not(:disabled):hover {
  background-color: var(--color-background-mute);
  border-color: var(--color-border-hover);
}

.btn-danger {
    background-color: var(--c-red-50);
    color: var(--c-red-700);
    border-color: var(--c-red-100);
}

.btn-danger:not(:disabled):hover {
    background-color: var(--c-red-100);
    border-color: var(--c-red-200);
}

/* Scrollbar */
.list-items::-webkit-scrollbar {
  width: 6px;
}
.list-items::-webkit-scrollbar-track {
  background: transparent;
}
.list-items::-webkit-scrollbar-thumb {
  background: var(--c-slate-300);
  border-radius: 3px;
}
.list-items::-webkit-scrollbar-thumb:hover {
  background: var(--c-slate-400);
}

/* Transition */
.list-collapse-enter-active,
.list-collapse-leave-active {
  transition: all 0.3s ease-in-out;
  max-height: 250px;
}

.list-collapse-enter-from,
.list-collapse-leave-to {
  opacity: 0;
  transform: translateY(-10px);
  max-height: 0;
}
</style>
