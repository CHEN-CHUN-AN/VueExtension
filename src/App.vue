<script setup>
import { ref, onMounted, watch, nextTick } from 'vue';
import canvasConfetti from 'canvas-confetti';
import History from './components/History.vue';
import DrawSettings from './components/DrawSettings.vue';
import ParticipantList from './components/ParticipantList.vue';
import DrawExecution from './components/DrawExecution.vue';
import GroupDisplay from './components/GroupDisplay.vue';

// 應用狀態
const mode = ref('number'); // 'number' or 'list'
const totalStudents = ref(0);
const numberList = ref([]);
const nameList = ref([]);
const nameInput = ref('');
const drawnItems = ref([]);
const isDrawing = ref(false);
const drawCount = ref(1);
const currentList = ref([]);
const history = ref([]);
const groups = ref([]);
const groupCount = ref(2);

// 新增UI狀態變數
const isGenerating = ref(false);
const isImporting = ref(false);
const showSuccess = ref(false);
const showError = ref(false);
const successMessage = ref('');
const errorMessage = ref('');
const isHovering = ref(false);
const isListVisible = ref(true);

// 從本地存儲加載數據
onMounted(async () => {
  try {
    if (typeof chrome !== 'undefined' && chrome.storage) {
      const stored = await chrome.storage.local.get(['history', 'lastMode', 'lastNumberList', 'lastNameList']);

      if (stored.history && Array.isArray(stored.history)) {
        history.value = stored.history;
      } else {
        history.value = [];
      }

      if (stored.lastMode) mode.value = stored.lastMode;

      if (stored.lastNumberList && Array.isArray(stored.lastNumberList) && stored.lastMode === 'number') {
        numberList.value = stored.lastNumberList;
        totalStudents.value = stored.lastNumberList.length;
      }

      if (stored.lastNameList && Array.isArray(stored.lastNameList) && stored.lastMode === 'list') {
        nameList.value = stored.lastNameList;
      }
    } else {
      const storedData = localStorage.getItem('lotteryData');
      if (storedData) {
        const stored = JSON.parse(storedData);

        if (stored.history && Array.isArray(stored.history)) {
          history.value = stored.history;
        }

        if (stored.lastMode) mode.value = stored.lastMode;

        if (stored.lastNumberList && Array.isArray(stored.lastNumberList) && stored.lastMode === 'number') {
          numberList.value = stored.lastNumberList;
          totalStudents.value = stored.lastNumberList.length;
        }

        if (stored.lastNameList && Array.isArray(stored.lastNameList) && stored.lastMode === 'list') {
          nameList.value = stored.lastNameList;
        }
      }
    }
    updateCurrentList();
  } catch (error) {
    console.log('Storage not available, using default values');
    history.value = [];
    numberList.value = [];
    nameList.value = [];
    currentList.value = [];
    drawnItems.value = [];
  }
});

// 保存到本地存儲
const saveToStorage = async () => {
  try {
    const data = {
      history: history.value,
      lastMode: mode.value,
      lastNumberList: numberList.value,
      lastNameList: nameList.value
    };

    if (typeof chrome !== 'undefined' && chrome.storage) {
      await chrome.storage.local.set(data);
    } else {
      localStorage.setItem('lotteryData', JSON.stringify(data));
    }
  } catch (error) {
    console.log('Storage not available');
  }
};

// 監聽模式變化
watch(mode, () => {
  updateCurrentList();
  saveToStorage();
  groups.value = [];
  drawnItems.value = [];
  history.value = [];
});

// 更新當前列表
const updateCurrentList = () => {
  if (mode.value === 'number') {
    currentList.value = [...numberList.value];
  } else {
    currentList.value = [...nameList.value];
  }
};

// 改進的成功訊息顯示
const showSuccessMessage = (message) => {
  successMessage.value = message;
  showSuccess.value = true;
  setTimeout(() => {
    showSuccess.value = false;
  }, 3500);
};

// 改進的錯誤訊息顯示
const showErrorMessage = (message) => {
  errorMessage.value = message;
  showError.value = true;
  setTimeout(() => {
    showError.value = false;
  }, 4000);
};

// 數字模式：生成名單
const generateNumberList = async () => {
  if (totalStudents.value > 0) {
    isGenerating.value = true;

    // 添加生成動畫延遲
    await new Promise(resolve => setTimeout(resolve, 800));

    numberList.value = Array.from({ length: totalStudents.value }, (_, i) => i + 1);
    updateCurrentList();
    drawnItems.value = [];
    saveToStorage();

    isGenerating.value = false;
    showSuccessMessage(`🎯 成功生成 ${totalStudents.value} 個號碼！`);
  }
};

// 名單模式：添加姓名
const addNames = async () => {
  if (!nameInput.value.trim()) {
    showErrorMessage('❌ 請輸入名單內容！');
    return;
  }

  const names = nameInput.value.split('\n')
    .map(name => name.trim())
    .filter(name => name.length > 0);

  if (names.length === 0) {
    showErrorMessage('❌ 沒有找到有效的姓名！');
    return;
  }

  const originalCount = nameList.value.length;
  nameList.value = [...new Set([...nameList.value, ...names])];
  const newCount = nameList.value.length;
  const addedCount = newCount - originalCount;

  nameInput.value = '';
  updateCurrentList();
  saveToStorage();

  showSuccessMessage(`✅ 成功添加 ${addedCount} 個新名單！目前總共有 ${newCount} 個名單。`);
};



const handleCSVFile = async (event) => {
  const file = event.target.files[0];
  if (!file) return;

  if (!file.name.toLowerCase().includes('.csv') && !file.name.toLowerCase().includes('.txt')) {
    showErrorMessage('❌ 請選擇 CSV 或 TXT 文件！');
    return;
  }

  isImporting.value = true;

  const reader = new FileReader();
  reader.onload = async (e) => {
    try {
      let csv = e.target.result;
      csv = csv.replace(/^\uFEFF/, '').replace(/^\ufeff/, '').replace(/^\u00EF\u00BB\u00BF/, '');

      let lines = csv.split(/\r\n|\r|\n/);
      if (lines.length === 1 && csv.includes(' ')) {
        lines = csv.split(/\s+/);
      }

      const names = [];
      lines.forEach((line) => {
        let name = line.trim();
        if (name && name.length > 0) {
          if (name.includes(',')) {
            name = name.split(',')[0].trim();
          }
          name = name.replace(/^["'`\s]+|["'`\s]+$/g, '');

          if (name &&
              name.length > 0 &&
              name.length <= 10 &&
              name !== '姓名' &&
              name !== 'name' &&
              name !== 'Name' &&
              name !== 'NAME' &&
              name !== '名字' &&
              !/^[\s\t\r\n]*$/.test(name)) {
            names.push(name);
          }
        }
      });

      if (names.length > 0) {
        const originalCount = nameList.value.length;
        const allNames = [...nameList.value, ...names];
        const uniqueNames = [...new Set(allNames)];
        nameList.value = uniqueNames;

        updateCurrentList();
        saveToStorage();

        const newCount = uniqueNames.length;
        const addedCount = newCount - originalCount;

        showSuccessMessage(`📥 CSV匯入成功！新增了 ${addedCount} 個名單，目前總共有 ${newCount} 個不重複的名單。`);
      } else {
        showErrorMessage('❌ CSV文件中沒有找到有效的名單數據！請確認文件格式正確。');
      }
    } catch (error) {
      showErrorMessage('❌ CSV文件讀取失敗！請檢查文件是否損壞或格式不正確。');
    } finally {
      isImporting.value = false;
    }
  };

  reader.onerror = () => {
    showErrorMessage('❌ 文件讀取失敗！請重試。');
    isImporting.value = false;
  };

  await new Promise(resolve => setTimeout(resolve, 500));
  reader.readAsText(file, 'UTF-8');
  event.target.value = '';
};

// 下載範例 CSV 文件
const downloadSampleCSV = () => {
  const sampleData = [
    ['張三'],
    ['李四'],
    ['王五'],
    ['趙六'],
    ['錢七'],
    ['孫八'],
    ['周九'],
    ['吳十']
  ];

  const csvContent = sampleData.map(row => row.join(',')).join('\n');
  const blob = new Blob(['\uFEFF' + csvContent], { type: 'text/csv;charset=utf-8;' });
  const link = document.createElement('a');

  if (link.download !== undefined) {
    const url = URL.createObjectURL(blob);
    link.setAttribute('href', url);
    link.setAttribute('download', '名單範例.csv');
    link.style.visibility = 'hidden';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }
};



// 抽獎動畫
const drawItems = () => {
  if (currentList.value.length === 0 || isDrawing.value || drawCount.value > currentList.value.length) {
    return;
  }

  isDrawing.value = true;
  drawnItems.value = [];

  let animationInterval;
  let count = 0;
  const maxCount = 35; // 增加動畫時間

  animationInterval = setInterval(() => {
    drawnItems.value = [];
    const tempAnimationList = [...currentList.value];

    for (let i = 0; i < Math.min(drawCount.value, tempAnimationList.length); i++) {
      const randomIndex = Math.floor(Math.random() * tempAnimationList.length);
      drawnItems.value.push(tempAnimationList[randomIndex]);
      tempAnimationList.splice(randomIndex, 1);
    }

    count++;
    if (count >= maxCount) {
      clearInterval(animationInterval);

      const finalDrawn = [];
      const tempList = [...currentList.value];

      for (let i = 0; i < drawCount.value && tempList.length > 0; i++) {
        const randomIndex = Math.floor(Math.random() * tempList.length);
        finalDrawn.push(tempList[randomIndex]);
        tempList.splice(randomIndex, 1);
      }

      drawnItems.value = [...finalDrawn];

      finalDrawn.forEach(item => {
        const index = currentList.value.indexOf(item);
        if (index > -1) {
          currentList.value.splice(index, 1);
          if (mode.value === 'number') {
            const numIndex = numberList.value.indexOf(item);
            if (numIndex > -1) numberList.value.splice(numIndex, 1);
          } else {
            const nameIndex = nameList.value.indexOf(item);
            if (nameIndex > -1) nameList.value.splice(nameIndex, 1);
          }
        }
      });

      history.value.unshift({
        timestamp: new Date().toLocaleString(),
        mode: mode.value,
        items: [...finalDrawn],
        drawCount: drawCount.value
      });

      isDrawing.value = false;
      triggerConfetti();
      saveToStorage();
    }
  }, 85); // 稍微減慢動畫速度
};

// 紙花動畫
const triggerConfetti = () => {
  const duration = 2500; // 減少持續時間
  const end = Date.now() + duration;

  (function frame() {
    canvasConfetti({
      particleCount: 20, // 減少紙花數量
      spread: 70,
      origin: { x: Math.random(), y: Math.random() - 0.2 },
      colors: ['#ff0000', '#00ff00', '#0000ff', '#ffff00', '#ff00ff', '#00ffff']
    });

    if (Date.now() < end) {
      requestAnimationFrame(frame);
    }
  }());
};

// 清空名單
const clearList = () => {
  if (mode.value === 'number') {
    totalStudents.value = 0;
    numberList.value = [];
  } else {
    nameList.value = [];
    nameInput.value = '';
  }
  currentList.value = [];
  drawnItems.value = [];
  saveToStorage();
  showSuccessMessage('🧹 名單已清空！');
};

// 手動刪除項目
const removeItem = (item) => {
  const index = currentList.value.indexOf(item);
  if (index > -1) {
    currentList.value.splice(index, 1);
    if (mode.value === 'number') {
      const numIndex = numberList.value.indexOf(item);
      if (numIndex > -1) numberList.value.splice(numIndex, 1);
    } else {
      const nameIndex = nameList.value.indexOf(item);
      if (nameIndex > -1) nameList.value.splice(nameIndex, 1);
    }
    saveToStorage();
  }
};

// 清空歷史記錄
const clearHistory = () => {
  history.value = [];
  saveToStorage();
  showSuccessMessage('🗑️ 歷史記錄已清空！');
};

// 恢復歷史項目到名單
const restoreFromHistory = (historyItem) => {
  historyItem.items.forEach(item => {
    if (mode.value === historyItem.mode) {
      if (mode.value === 'number') {
        if (!numberList.value.includes(item)) {
          numberList.value.push(item);
        }
      } else {
        if (!nameList.value.includes(item)) {
          nameList.value.push(item);
        }
      }
    }
  });
  updateCurrentList();
  saveToStorage();
  showSuccessMessage('↩️ 已恢復到名單！');
};

// 切換列表顯示
const toggleListVisibility = () => {
  isListVisible.value = !isListVisible.value;
};

// 分組功能
const groupParticipants = () => {
  if (currentList.value.length < groupCount.value) {
    showErrorMessage('❌ 人數不足，無法分組！');
    return;
  }

  const shuffled = [...currentList.value].sort(() => 0.5 - Math.random());
  const result = [];
  for (let i = 0; i < groupCount.value; i++) {
    result.push([]);
  }

  shuffled.forEach((item, index) => {
    result[index % groupCount.value].push(item);
  });

  groups.value = result;
  showSuccessMessage('🎉 分組完成！');
};
</script>

<template>
  <div class="app-container">
    <!-- 成功提示 -->
    <div :class="['notification', 'success', { show: showSuccess }]">
      <div class="notification-content">
        <span class="notification-icon">✨</span>
        <span class="notification-text">{{ successMessage }}</span>
      </div>
    </div>

    <!-- 錯誤提示 -->
    <div :class="['notification', 'error', { show: showError }]">
      <div class="notification-content">
        <span class="notification-icon">⚠️</span>
        <span class="notification-text">{{ errorMessage }}</span>
      </div>
    </div>

    <!-- 主容器 -->
    <div class="main-container">
      <!-- 標題區域 -->
      <div class="header">
        <h1 class="title">抽獎小幫手</h1>
        <p class="subtitle">簡簡單單方方便便就抽獎！</p>
      </div>

      <!-- 主要內容區域 -->
      <div class="content-area">
        <!-- 抽獎設定區域 -->
        <div class="section-card">
          <h2 class="section-title">抽獎設定</h2>
          <DrawSettings
            v-model:mode="mode"
            v-model:totalStudents="totalStudents"
            v-model:nameInput="nameInput"
            :is-generating="isGenerating"
            :is-importing="isImporting"
            @generate-number-list="generateNumberList"
            @download-sample-csv="downloadSampleCSV"
            @add-names="addNames"
            @handle-csv-file="handleCSVFile"
          />
        </div>

        <!-- 抽獎控制與結果 -->
        <div class="section-card">
          <h2 class="section-title">執行抽獎</h2>
          <DrawExecution
            v-model:drawCount="drawCount"
            :current-list="currentList"
            :is-drawing="isDrawing"
            :drawn-items="drawnItems"
            @draw-items="drawItems"
          />
        </div>

        <!-- 當前名單與分組 -->
        <div class="section-card">
           <ParticipantList
            :current-list="currentList"
            :is-list-visible="isListVisible"
            :groups="groups"
            v-model:groupCount="groupCount"
            @toggle-list-visibility="toggleListVisibility"
            @clear-list="clearList"
            @remove-item="removeItem"
            @group-participants="groupParticipants"
          />
          <GroupDisplay :groups="groups" />
        </div>

        <!-- 歷史記錄區域 -->
        <History
          :history="history"
          @clear-history="clearHistory"
          @restore-from-history="restoreFromHistory"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  background: var(--color-background);
  padding: 1rem 0;
}

.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
  padding: 1rem 1.25rem;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  background: var(--c-white);
  color: var(--c-slate-800);
  border-left: 4px solid;
  display: flex;
  align-items: center;
  transform: translateX(120%);
  transition: transform 0.4s ease-out;
}

.notification.show {
  transform: translateX(0);
}

.notification.success {
  border-color: var(--c-green-600);
  background: var(--c-green-50);
  color: var(--c-green-700);
}

.notification.error {
  border-color: var(--c-red-600);
  background: var(--c-red-50);
  color: var(--c-red-700);
}

.notification-content {
  display: flex;
  align-items: center;
}

.notification-icon {
  font-size: 1.25rem;
  margin-right: 0.75rem;
}

.notification-text {
  font-size: 0.9rem;
  font-weight: 500;
}

.main-container {
  background: var(--c-white);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
  overflow: hidden;
  border: 1px solid var(--color-border);
}

.header {
  background: var(--c-slate-800);
  color: var(--c-white);
  text-align: center;
  padding: 2rem 1.5rem;
  position: relative;
}

.title {
  font-size: 1.75rem;
  font-weight: 800;
  margin-bottom: 0.5rem;
}

.subtitle {
  font-size: 1rem;
  opacity: 0.8;
  font-weight: 400;
}

.content-area {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

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
  margin-bottom: 1.5rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--color-border);
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

@media (max-width: 768px) {
  .app-container {
    padding: 0;
  }
  .main-container {
    border-radius: 0;
    border: none;
    box-shadow: none;
  }
  .content-area {
    padding: 1.5rem 1rem;
  }
  .header {
    padding: 1.5rem 1rem;
  }
  .title {
    font-size: 1.5rem;
  }
  .notification {
    top: 10px;
    right: 10px;
    left: 10px;
    border-radius: var(--radius-md);
  }
}
</style>
