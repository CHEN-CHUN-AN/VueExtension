<script setup>
import { ref } from 'vue';

defineProps({
  drawCount: { type: Number, required: true },
  currentList: { type: Array, required: true },
  isDrawing: { type: Boolean, required: true },
  drawnItems: { type: Array, required: true },
});

const emit = defineEmits([
  'update:drawCount',
  'draw-items',
]);

const onDrawCountChange = (event) => {
  emit('update:drawCount', Number(event.target.value));
};

</script>

<template>
  <div class="draw-section">
    <div class="draw-controls">
      <div class="control-group">
        <label class="control-label" for="draw-count-input">抽取人數：</label>
        <input
          id="draw-count-input"
          type="number"
          :value="drawCount"
          @input="onDrawCountChange"
          min="1"
          :max="currentList.length"
          class="draw-count-input"
        />
      </div>
      <button
        @click="emit('draw-items')"
        :disabled="currentList.length === 0 || isDrawing || drawCount > currentList.length"
        class="btn btn-primary btn-draw"
      >
        <span v-if="isDrawing" class="draw-animation">🎲</span>
        {{ isDrawing ? '抽獎中...' : '🚀 開始抽獎' }}
      </button>
    </div>

    <!-- 抽獎結果顯示 -->
    <Transition name="result-fade">
      <div v-if="drawnItems.length > 0" class="result-section">
        <h3 class="result-title">🎉 中獎結果</h3>
        <div class="result-items">
          <TransitionGroup name="result-item-pop">
            <div
              v-for="(item, index) in drawnItems"
              :key="item"
              class="result-item"
              :class="{ 'drawing': isDrawing }"
              :style="{ transitionDelay: `${index * 50}ms` }"
            >
              {{ item }}
            </div>
          </TransitionGroup>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.draw-section {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.draw-controls {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.control-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.control-label {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--color-heading);
}

.draw-count-input {
  padding: 0.75rem 1rem;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: 1rem;
  background: var(--c-white);
  transition: border-color 0.2s, box-shadow 0.2s;
  width: 100px;
}

.draw-count-input:focus {
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
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

.btn-draw {
    padding-top: 1rem;
    padding-bottom: 1rem;
    font-size: 1.1rem;
}

.draw-animation {
  animation: bounce 0.6s ease-in-out infinite alternate;
  display: inline-block;
  margin-right: 0.5rem;
}

@keyframes bounce {
  to { transform: scale(1.2) rotate(15deg); }
}

/* Result Section */
.result-section {
  margin-top: 1rem;
  padding: 1.5rem;
  background: var(--c-amber-50);
  border-radius: var(--radius-lg);
  border: 1px solid var(--c-amber-300);
}

.result-title {
  font-size: 1.25rem;
  color: var(--c-amber-800, #92400e);
  font-weight: 700;
  text-align: center;
  margin-bottom: 1rem;
}

.result-items {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  justify-content: center;
}

.result-item {
  padding: 0.75rem 1.25rem;
  background: var(--c-white);
  border-radius: var(--radius-md);
  font-size: 1.1rem;
  font-weight: 700;
  color: var(--color-primary);
  border: 1px solid var(--color-primary-bg-hover);
  box-shadow: var(--shadow-sm);
}

.result-item.drawing {
  animation: pulse 0.8s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

/* Transitions */
.result-fade-enter-active,
.result-fade-leave-active {
  transition: opacity 0.3s ease;
}
.result-fade-enter-from,
.result-fade-leave-to {
  opacity: 0;
}

.result-item-pop-enter-active {
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.result-item-pop-leave-active {
  transition: all 0.3s ease-in;
}
.result-item-pop-enter-from,
.result-item-pop-leave-to {
  opacity: 0;
  transform: scale(0.5) translateY(20px);
}

@media (max-width: 480px) {
    .control-group {
        flex-direction: column;
        align-items: stretch;
    }
    .draw-count-input {
        width: 100%;
    }
}
</style>
