<script setup>
defineProps({
  history: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['clear-history', 'restore-from-history']);

const clearHistory = () => {
  emit('clear-history');
};

const restoreFromHistory = (item) => {
  emit('restore-from-history', item);
};
</script>

<template>
  <div class="section-card">
    <div class="history-header">
        <h2 class="section-title">📚 歷史記錄</h2>
        <button
          @click="clearHistory"
          :disabled="history.length === 0"
          class="btn btn-sm btn-secondary"
        >
          🗑️ 清空歷史
        </button>
    </div>

    <div class="history-section">
      <div v-if="history.length === 0" class="empty-history">
        <p class="empty-text">暫無抽獎記錄</p>
      </div>

      <div v-else class="history-items">
        <div
          v-for="(item, index) in history"
          :key="index"
          class="history-item"
        >
          <div class="history-item-main">
            <div class="history-info">
              <div class="history-time">⏰ {{ item.timestamp }}</div>
              <div class="history-details">
                <span class="detail-tag">{{ item.mode === 'number' ? '🔢 數字' : '📝 名單' }}</span>
                <span class="detail-tag">抽 {{ item.drawCount }} 項</span>
              </div>
            </div>

            <div class="history-results">
              <span
                v-for="(result, idx) in item.items"
                :key="idx"
                class="history-result"
              >
                {{ result }}
              </span>
            </div>
          </div>
          <button
            @click="restoreFromHistory(item)"
            class="btn btn-sm btn-green-secondary"
          >
            ↩️ 恢復
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.section-card {
  background: var(--c-white);
  border-radius: var(--radius-lg);
  padding: 1.5rem;
  border: 1px solid var(--color-border);
}

.section-title {
  font-size: 1.25rem;
  color: var(--color-heading);
  font-weight: 700;
  margin: 0;
  padding: 0;
  border: none;
}

.history-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
}

/* History Section */
.empty-history {
  text-align: center;
  padding: 3rem 1rem;
  color: var(--color-text-muted);
  background-color: var(--color-background-soft);
  border-radius: var(--radius-md);
}

.empty-text {
  font-size: 1rem;
  font-weight: 500;
}

.history-items {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  max-height: 400px;
  overflow-y: auto;
  padding-right: 0.5rem; /* For scrollbar */
}

.history-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  background: var(--color-background-soft);
  border-radius: var(--radius-md);
  padding: 1rem;
  border: 1px solid var(--color-border);
  transition: background-color 0.2s ease;
}

.history-item:hover {
    background-color: var(--color-background-mute);
}

.history-item-main {
    flex-grow: 1;
}

.history-info {
  margin-bottom: 0.75rem;
}

.history-time {
  font-size: 0.8rem;
  color: var(--color-text-muted);
  margin-bottom: 0.5rem;
}

.history-details {
    display: flex;
    gap: 0.5rem;
}

.detail-tag {
    font-size: 0.75rem;
    font-weight: 500;
    padding: 0.2rem 0.5rem;
    border-radius: var(--radius-full);
    background-color: var(--color-background-mute);
    color: var(--color-text);
}

.history-results {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.history-result {
  padding: 0.25rem 0.75rem;
  background: var(--c-blue-50);
  color: var(--c-blue-600);
  border-radius: var(--radius-md);
  font-size: 0.9rem;
  font-weight: 600;
  border: 1px solid var(--c-blue-100);
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
  white-space: nowrap;
}

.btn-sm {
    padding: 0.4rem 0.8rem;
    font-size: 0.8rem;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
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

.btn-green-secondary {
    background-color: var(--c-green-50);
    color: var(--c-green-700);
    border-color: var(--c-green-100);
}

.btn-green-secondary:not(:disabled):hover {
    background-color: var(--c-green-100);
    border-color: var(--c-green-200, #a7f3d0);
}

/* Scrollbar */
.history-items::-webkit-scrollbar {
  width: 6px;
}
.history-items::-webkit-scrollbar-track {
  background: transparent;
}
.history-items::-webkit-scrollbar-thumb {
  background: var(--c-slate-300);
  border-radius: 3px;
}
.history-items::-webkit-scrollbar-thumb:hover {
  background: var(--c-slate-400);
}
</style>