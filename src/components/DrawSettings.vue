<script setup>
import { ref } from 'vue';

defineProps({
  mode: { type: String, required: true },
  totalStudents: { type: Number, required: true },
  nameInput: { type: String, required: true },
  isGenerating: { type: Boolean, required: true },
  isImporting: { type: Boolean, required: true },
});

const emit = defineEmits([
  'update:mode',
  'update:totalStudents',
  'update:nameInput',
  'generate-number-list',
  'download-sample-csv',
  'import-csv',
  'add-names',
  'handle-csv-file',
]);

const fileInput = ref(null);

const handleFileChange = (event) => {
  emit('handle-csv-file', event);
};

const triggerImport = () => {
  fileInput.value?.click();
};

const onModeChange = (newMode) => {
  emit('update:mode', newMode);
};

const onTotalStudentsChange = (event) => {
  emit('update:totalStudents', Number(event.target.value));
};

const onNameInputChange = (event) => {
  emit('update:nameInput', event.target.value);
};

</script>

<template>
  <div>
    <!-- 模式選擇 -->
    <div class="mode-selector">
      <label class="mode-option">
        <input type="radio" :checked="mode === 'number'" @change="onModeChange('number')" value="number" name="mode-selector" />
        <span class="mode-label">🔢 數字模式</span>
      </label>
      <label class="mode-option">
        <input type="radio" :checked="mode === 'list'" @change="onModeChange('list')" value="list" name="mode-selector" />
        <span class="mode-label">📝 名單模式</span>
      </label>
    </div>

    <!-- 數字模式設定 -->
    <div v-if="mode === 'number'" class="input-section">
      <div class="input-group">
        <label class="input-label" for="total-students-input">總人數：</label>
        <input
          id="total-students-input"
          type="number"
          :value="totalStudents"
          @input="onTotalStudentsChange"
          min="1"
          max="1000"
          class="number-input"
          placeholder="輸入總人數"
        />
        <button
          @click="emit('generate-number-list')"
          :disabled="totalStudents <= 0 || isGenerating"
          class="btn btn-primary"
        >
          <span v-if="isGenerating" class="loading-spinner"></span>
          {{ isGenerating ? '生成中...' : '🎯 產生名單' }}
        </button>
      </div>
    </div>

    <!-- 名單模式設定 -->
    <div v-if="mode === 'list'" class="input-section">
      <div class="input-group">
        <label class="input-label">從檔案匯入：</label>
        <div class="csv-controls">
          <button @click="emit('download-sample-csv')" class="btn btn-amber">
            📥 下載範例
          </button>
          <button
            @click="triggerImport"
            :disabled="isImporting"
            class="btn btn-secondary"
          >
            <span v-if="isImporting" class="loading-spinner"></span>
            {{ isImporting ? '匯入中...' : '📤 匯入 CSV/TXT' }}
          </button>
          <input
            ref="fileInput"
            type="file"
            accept=".csv,.txt"
            @change="handleFileChange"
            style="display: none"
          />
        </div>
      </div>

      <div class="input-group">
        <label class="input-label" for="name-input-area">或手動輸入名單：</label>
        <textarea
          id="name-input-area"
          :value="nameInput"
          @input="onNameInputChange"
          placeholder="請輸入名單，一行一個人員名稱..."
          class="name-textarea"
          rows="6"
        ></textarea>
        <button
          @click="emit('add-names')"
          :disabled="!nameInput.trim()"
          class="btn btn-primary"
        >
          ➕ 添加到名單
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* General component styling */
.input-section {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.input-label {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--color-heading);
}

/* Mode Selector */
.mode-selector {
  display: grid;
  grid-template-columns: 1fr 1fr;
  background-color: var(--color-background-soft);
  padding: 0.25rem;
  border-radius: var(--radius-md);
  margin-bottom: 1rem;
}

.mode-option {
  position: relative;
}

.mode-option input[type="radio"] {
  position: absolute;
  opacity: 0;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  cursor: pointer;
}

.mode-label {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0.75rem 1rem;
  border-radius: var(--radius-sm);
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--color-text-muted);
  transition: background-color 0.2s ease-in-out, color 0.2s ease-in-out;
  text-align: center;
}

.mode-option input[type="radio"]:checked + .mode-label {
  background-color: var(--c-white);
  color: var(--color-primary);
  box-shadow: var(--shadow-sm);
}

.mode-option input[type="radio"]:focus-visible + .mode-label {
    outline: 2px solid var(--color-primary);
    outline-offset: 2px;
}

/* Input Fields */
.number-input,
.name-textarea {
  padding: 0.75rem 1rem;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: 1rem;
  background: var(--c-white);
  transition: border-color 0.2s, box-shadow 0.2s;
  width: 100%;
}

.number-input:focus,
.name-textarea:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px var(--c-blue-100);
}

.name-textarea {
  resize: vertical;
  min-height: 120px;
  font-family: var(--font-sans);
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

.btn-secondary {
  background-color: var(--color-background-soft);
  color: var(--color-text);
  border-color: var(--color-border);
}

.btn-secondary:not(:disabled):hover {
  background-color: var(--color-background-mute);
  border-color: var(--color-border-hover);
}

.btn-amber {
    background-color: var(--c-amber-100);
    color: var(--c-amber-700);
    border-color: var(--c-amber-200);
}

.btn-amber:not(:disabled):hover {
    background-color: var(--c-amber-200);
    border-color: var(--c-amber-300);
}

.csv-controls {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.loading-spinner {
  width: 1rem;
  height: 1rem;
  border: 2px solid currentColor;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-right: 0.5rem;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

@media (max-width: 480px) {
    .mode-selector {
        grid-template-columns: 1fr;
    }
    .csv-controls {
        flex-direction: column;
    }
    .btn {
        width: 100%;
    }
}
</style>
