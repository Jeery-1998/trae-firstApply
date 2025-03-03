<!--
 * @Author: allkan-guojing maguojing@allcam.com.cn
 * @Date: 2025-03-03 18:05:53
 * @LastEditors: allkan-guojing maguojing@allcam.com.cn
 * @LastEditTime: 2025-03-03 20:39:01
 * @FilePath: \Trae\my-vue-app\src\components\SettingsIcon.vue
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<script setup lang="ts">
import { ref, watch } from 'vue';

const props = defineProps<{
  initialSettings?: {
    prizes: string[];
    rotationSpeed: number;
    rotationDuration: number;
    wheelStyle: {
      pointerColor: string;
      saturation: number;
      lightness: number;
    };
    title: string;
  }
}>();

const isOpen = ref(false);
const prizes = ref(props.initialSettings?.prizes || ['一等奖', '二等奖', '三等奖', '四等奖']);
const rotationSpeed = ref(props.initialSettings?.rotationSpeed || 10);
const rotationDuration = ref(props.initialSettings?.rotationDuration || 5);
const wheelStyle = ref(props.initialSettings?.wheelStyle || {
  pointerColor: '#e91e63',
  saturation: 70,
  lightness: 75
});

const wheelTitle = ref(props.initialSettings?.title || '幸运转盘');
const showImportModal = ref(false);
const importText = ref('');

// 监听props变化，更新本地状态
watch(() => props.initialSettings, (newSettings) => {
  if (newSettings) {
    // 使用深拷贝确保数组和对象的完全更新
    prizes.value = JSON.parse(JSON.stringify(newSettings.prizes));
    rotationSpeed.value = Number(newSettings.rotationSpeed);
    rotationDuration.value = Number(newSettings.rotationDuration);
    wheelStyle.value = JSON.parse(JSON.stringify(newSettings.wheelStyle));
    wheelTitle.value = String(newSettings.title);
  }
}, { deep: true, immediate: true });

// 移除对本地状态的实时监听，改为在保存时触发更新
const saveSettings = () => {
  if (prizes.value.length < 2) {
    alert('至少需要2个奖项');
    return;
  }
  if (prizes.value.length > 50) {
    alert('奖项数量不能超过50个');
    return;
  }
  const newSettings = {
    prizes: JSON.parse(JSON.stringify(prizes.value)),
    rotationSpeed: Number(rotationSpeed.value),
    rotationDuration: Number(rotationDuration.value),
    wheelStyle: JSON.parse(JSON.stringify(wheelStyle.value)),
    title: String(wheelTitle.value)
  };
  emit('settingsUpdated', newSettings);
  isOpen.value = false;
};

const cancelSettings = () => {
  // 取消时重置为初始值
  if (props.initialSettings) {
    prizes.value = JSON.parse(JSON.stringify(props.initialSettings.prizes));
    rotationSpeed.value = Number(props.initialSettings.rotationSpeed);
    rotationDuration.value = Number(props.initialSettings.rotationDuration);
    wheelStyle.value = JSON.parse(JSON.stringify(props.initialSettings.wheelStyle));
    wheelTitle.value = String(props.initialSettings.title);
  }
  isOpen.value = false;
};

// 移除实时监听，改为只在保存时更新数据

const handleImport = () => {
  const items = importText.value
    .split(/[,，\n]/)
    .map(item => item.trim())
    .filter(item => item)
    .filter((item, index, self) => self.indexOf(item) === index); // 去除重复项

  if (items.length < 2) {
    alert('请至少输入2个不同的奖项');
    return;
  }
  if (items.length > 50) {
    alert('奖项数量不能超过50个');
    return;
  }

  prizes.value = items;
  showImportModal.value = false;
  importText.value = '';
};

const toggleSettings = () => {
  isOpen.value = !isOpen.value;
};

const addPrize = () => {
  if (prizes.value.length >= 50) {
    alert('最多只能添加50个奖项');
    return;
  }
  prizes.value.push(`奖项 ${prizes.value.length + 1}`);
};

const removePrize = (index: number) => {
  if (prizes.value.length <= 2) {
    alert('至少需要保留2个奖项');
    return;
  }
  prizes.value.splice(index, 1);
};

const emit = defineEmits(['settingsUpdated']);

// 移除重复定义的函数
</script>

<template>
  <div class="settings-container">
    <button class="settings-icon" @click="toggleSettings" title="设置">
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <circle cx="12" cy="12" r="3"></circle>
        <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path>
      </svg>
    </button>
    <div v-if="isOpen" class="settings-panel">
      <h2 class="settings-title">转盘设置</h2>
      <div class="settings-content">
        <div class="settings-group">
          <h4>标题设置</h4>
          <div class="setting-item">
            <input type="text" v-model="wheelTitle" placeholder="请输入转盘标题" class="title-input">
          </div>
        </div>
        <div class="settings-group">
          <h4>奖项管理</h4>
          <div class="prize-list-container">
            <div class="prize-list-actions">
              <button class="upload-btn" @click="showImportModal = true">批量导入</button>
            </div>
            <div class="prize-list">
              <div v-for="(prize, index) in prizes" :key="index" class="prize-item">
                <input v-model="prizes[index]" type="text" :placeholder="`奖项 ${index + 1}`">
                <button @click="removePrize(index)" class="remove-btn">删除</button>
              </div>
            </div>
            <button @click="addPrize" class="add-btn">添加奖项</button>
          </div>
        </div>
        
        <div class="settings-group">
          <h4>动画设置</h4>
          <div class="setting-item">
            <label>转盘速度</label>
            <input type="range" v-model="rotationSpeed" min="1" max="20" class="slider">
            <span>{{rotationSpeed}}</span>
          </div>
          <div class="setting-item">
            <label>旋转时间</label>
            <input type="range" v-model="rotationDuration" min="1" max="10" class="slider">
            <span>{{rotationDuration}}秒</span>
          </div>
        </div>

        <div class="settings-group">
          <h4>外观设置</h4>
          <div class="setting-item">
            <label>指针颜色</label>
            <input type="color" v-model="wheelStyle.pointerColor">
          </div>
          <div class="setting-item">
            <label>颜色饱和度</label>
            <input type="range" v-model="wheelStyle.saturation" min="0" max="100" class="slider">
            <span>{{wheelStyle.saturation}}%</span>
          </div>
          <div class="setting-item">
            <label>颜色亮度</label>
            <input type="range" v-model="wheelStyle.lightness" min="0" max="100" class="slider">
            <span>{{wheelStyle.lightness}}%</span>
          </div>
        </div>

        <div class="button-group">
          <button @click="saveSettings" class="save-btn">保存</button>
          <button @click="cancelSettings" class="cancel-btn">取消</button>
        </div>
      </div>
    </div>

    <!-- 批量导入模态框 -->
    <div v-if="showImportModal" class="modal-overlay">
      <div class="modal-content">
        <h3>批量导入奖项</h3>
        <p class="modal-tip">请输入奖项，每个奖项用逗号或换行分隔（2-50个）</p>
        <textarea
          v-model="importText"
          class="import-textarea"
          placeholder="例如：一等奖，二等奖，三等奖
或者：
一等奖
二等奖
三等奖"
        ></textarea>
        <div class="modal-buttons">
          <button @click="handleImport" class="save-btn">导入</button>
          <button @click="showImportModal = false" class="cancel-btn">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.settings-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
}

.settings-icon {
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  border-radius: 50%;
  transition: background-color 0.3s;
  color: #2c3e50;
}

.settings-icon:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

.settings-panel {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 10px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 16px;
  min-width: 200px;
}

.settings-panel h3 {
  margin: 0 0 12px 0;
  color: #2c3e50;
}
.settings-title{
  font-family: 'Courier New', Courier, monospace;
  font-weight: bolder;
}
.settings-content {
  color: #666;
}

.settings-group {
  margin-bottom: 20px;
}

.settings-group h4 {
  margin: 0 0 10px 0;
  color: #2c3e50;
  font-size: 16px;
}

.prize-list-container {
  margin-bottom: 10px;
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
}

.prize-list-actions {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 15px;
}

.upload-btn {
  display: inline-flex;
  align-items: center;
  padding: 8px 16px;
  background-color: #673ab7;
  color: white;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
  border: none;
  box-shadow: 0 2px 4px rgba(103, 58, 183, 0.2);
}

.upload-btn:hover {
  background-color: #5e35b1;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(103, 58, 183, 0.3);
}

.prize-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 15px;
  max-height: 300px;
  overflow-y: auto;
  padding: 5px 10px 5px 5px;
  background: white;
  border-radius: 6px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.05);
}

.prize-item {
  display: flex;
  gap: 12px;
  align-items: center;
  padding: 8px;
  background: #fff;
  border-radius: 6px;
  transition: all 0.3s ease;
  border: 1px solid #e0e0e0;
}

.prize-item:hover {
  border-color: #673ab7;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.prize-item input {
  flex: 1;
  padding: 10px;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  font-size: 14px;
  transition: all 0.3s ease;
  background: #f8f9fa;
}

.prize-item input:focus {
  outline: none;
  border-color: #673ab7;
  background: white;
  box-shadow: 0 0 0 2px rgba(103, 58, 183, 0.1);
}

.remove-btn {
  padding: 8px 16px;
  background-color: #ff5722;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
  box-shadow: 0 2px 4px rgba(255, 87, 34, 0.2);
}

.remove-btn:hover {
  background-color: #f4511e;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(255, 87, 34, 0.3);
}

.add-btn {
  width: 100%;
  padding: 12px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
  font-weight: 500;
  box-shadow: 0 2px 4px rgba(33, 150, 243, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.add-btn:hover {
  background-color: #1976d2;
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(33, 150, 243, 0.3);
}

.add-btn::before {
  content: '+';
  font-size: 18px;
  font-weight: bold;
  margin-right: 4px;
}

.title-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.prize-item {
  display: flex;
  gap: 10px;
}

.prize-item input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.setting-item {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}

.setting-item label {
  min-width: 80px;
}

.setting-item input[type="range"] {
  flex: 1;
}

.setting-item input[type="color"] {
  width: 50px;
  height: 30px;
  padding: 0;
  border: 1px solid #ddd;
}

.button-group {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.save-btn,
.cancel-btn,
.add-btn,
.remove-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.save-btn {
  background-color: #4caf50;
  color: white;
}

.cancel-btn {
  background-color: #f44336;
  color: white;
}

.add-btn {
  background-color: #2196f3;
  color: white;
  width: 100%;
}

.remove-btn {
  background-color: #ff5722;
  color: white;
}

.save-btn:hover,
.cancel-btn:hover,
.add-btn:hover,
.remove-btn:hover {
  opacity: 0.9;
}

/* 模态框样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 90%;
  max-width: 500px;
}

.modal-content h3 {
  margin: 0 0 16px 0;
  color: #2c3e50;
}

.modal-tip {
  color: #666;
  margin-bottom: 12px;
  font-size: 14px;
}

.import-textarea {
  width: 100%;
  height: 120px;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 16px;
  resize: vertical;
  font-size: 14px;
  line-height: 1.5;
}

.modal-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}
</style>