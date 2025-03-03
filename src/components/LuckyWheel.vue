<!--
 * @Author: allkan-guojing maguojing@allcam.com.cn
 * @Date: 2025-03-03 16:55:04
 * @LastEditors: allkan-guojing maguojing@allcam.com.cn
 * @LastEditTime: 2025-03-03 20:28:49
 * @FilePath: \Trae\my-vue-app\src\components\LuckyWheel.vue
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<template>
  <div class="lucky-wheel-container">
    
    <div class="wheel-section">
      <div class="pointer"></div>
      <canvas ref="wheelCanvas" width="400" height="400"></canvas>
    </div>
    <div class="control-panel">
      <div class="button-group">
        <button @click="startRotation" :disabled="isRotating || prizes.length < 2" class="start-btn">开始抽奖</button>
      </div>
    </div>

    
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed, nextTick } from 'vue';

const wheelCanvas = ref<HTMLCanvasElement | null>(null);
const isRotating = ref(false);
const result = ref('');
// 定义props和响应式引用
const props = defineProps<{
  prizes: string[];
  rotationSpeed: number;
  rotationDuration: number;
  wheelStyle: {
    pointerColor: string;
    saturation: number;
    lightness: number;
  };
  wheelTitle: string;
}>();

// 创建响应式引用
const prizes = ref<string[]>([]);
const rotationSpeed = ref(props.rotationSpeed);
const rotationDuration = ref(props.rotationDuration);
const wheelStyle = ref({ ...props.wheelStyle });
const wheelTitle = ref(props.wheelTitle);

// 监听动画相关props的变化
watch(() => props.rotationSpeed, (newSpeed) => {
  rotationSpeed.value = newSpeed;
}, { immediate: true });

watch(() => props.rotationDuration, (newDuration) => {
  rotationDuration.value = newDuration;
}, { immediate: true });

// 监听props变化并更新本地响应式数据
watch(() => props.prizes, (newPrizes) => {
  prizes.value = [...newPrizes];
  nextTick(() => {
    drawWheel();
  });
}, { immediate: true, deep: true });

watch(() => props.wheelStyle, (newStyle) => {
  wheelStyle.value = { ...newStyle };
  nextTick(() => {
    drawWheel();
  });
}, { immediate: true, deep: true });

watch(() => props.wheelTitle, (newTitle) => {
  wheelTitle.value = newTitle;
  nextTick(() => {
    drawWheel();
  });
}, { immediate: true });
const emit = defineEmits(['update:prizes', 'update:rotationSpeed', 'update:rotationDuration', 'update:wheelStyle', 'update:wheelTitle']);


// 预设颜色数组替换为动态生成颜色的函数
const getSliceColor = (index: number, total: number) => {
  // 使用HSL颜色空间，通过索引计算色相值
  const hue = (index * (360 / total)) % 360;
  // 使用配置的饱和度和亮度
  return `hsl(${hue}, ${wheelStyle.value.saturation}%, ${wheelStyle.value.lightness}%)`;
};

// 绘制转盘
const drawWheel = () => {
  const canvas = wheelCanvas.value;
  if (!canvas) return;
  
  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  // 清空画布
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  const centerX = canvas.width / 2;
  const centerY = canvas.height / 2;
  const radius = Math.min(centerX, centerY) - 10;

  const sliceAngle = (Math.PI * 2) / prizes.value.length;

  for (let i = 0; i < prizes.value.length; i++) {
    ctx.save();
    ctx.beginPath();
    ctx.moveTo(centerX, centerY);
    ctx.arc(centerX, centerY, radius, i * sliceAngle, (i + 1) * sliceAngle);
    ctx.closePath();

    // 使用动态生成的颜色
    ctx.fillStyle = getSliceColor(i, prizes.value.length);
    ctx.fill();
    ctx.restore();

    // 绘制文字
    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.rotate(i * sliceAngle + sliceAngle / 2);
    ctx.textAlign = 'right';
    ctx.fillStyle = '#000';
    ctx.font = 'bold 14px Arial';
    ctx.fillStyle = '#333333';
    ctx.fillText(prizes.value[i], radius - 30, 0);
    ctx.restore();
  }
};

// 监听奖项和样式变化，重新绘制转盘
watch([prizes, wheelStyle], () => {
  console.log(prizes,'prizes');
  
  drawWheel();
}, { deep: true });

// 监听转盘标题变化
watch(wheelTitle, () => {
  drawWheel();
});

// 开始旋转
const startRotation = () => {
  if (isRotating.value || prizes.value.length < 2) return;
  
  isRotating.value = true;
  result.value = '';
  
  // 随机选择一个奖项
  const selectedIndex = Math.floor(Math.random() * prizes.value.length);
  // 计算每个扇形的角度
  const sliceAngle = 360 / prizes.value.length;
  // 基础圈数（8圈）加上随机的额外圈数（0-2圈），使每次旋转的圈数不完全相同
  const baseRotations = 8;
  const extraRotations = Math.random() * 2;
  const totalRotations = baseRotations + extraRotations;
  
  // 修正角度计算：确保指针指向选中奖项的中心
  const targetAngle = totalRotations * 360 + selectedIndex * sliceAngle;
  
  let currentAngle = 0;
  let startTime = performance.now();
  const duration = rotationDuration.value * 1000;
  
  const rotate = (currentTime: number) => {
    if (!wheelCanvas.value) return;
    
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // 使用改进的缓动函数
    const easeOut = (t: number) => {
      return 1 - Math.pow(1 - t, 3);
    };
    
    const currentProgress = easeOut(progress);
    currentAngle = currentProgress * targetAngle;
    
    // 应用旋转变换
    const speedFactor = (rotationSpeed.value / 10);
    const adjustedAngle = currentAngle * speedFactor;
    wheelCanvas.value.style.transform = `rotate(${adjustedAngle}deg)`;
    
    if (progress < 1) {
      requestAnimationFrame(rotate);
    } else {
      // 确保最终角度精确对应选中的奖项
      wheelCanvas.value.style.transform = `rotate(${targetAngle * speedFactor}deg)`;
      isRotating.value = false;
      result.value = prizes.value[selectedIndex];
    }
  };
  
  requestAnimationFrame((time) => {
    startTime = time;
    rotate(time);
  });
};

onMounted(() => {
  drawWheel();
});



</script>

<style scoped>
.lucky-wheel-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px;
  padding: 30px;
  background: linear-gradient(to bottom, #f8f9fa, #e9ecef);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.wheel-title {
  font-size: 32px;
  font-weight: bold;
  color: #2c3e50;
  margin: 0;
  padding: 15px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.text-input {
  flex: 1;
  padding: 12px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 16px;
  transition: all 0.3s ease;
}

.text-input:focus {
  border-color: #2196F3;
  box-shadow: 0 0 0 3px rgba(33, 150, 243, 0.2);
  outline: none;
}

.wheel-section {
  position: relative;
  width: 400px;
  height: 400px;
  padding: 20px;
  background: white;
  border-radius: 50%;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  align-items: center;
}

.pointer {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%) rotate(180deg);
  width: 40px;
  height: 40px;
  background: v-bind(wheelStyle.pointerColor);
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  filter: drop-shadow(0 4px 6px rgba(0, 0, 0, 0.2));
  z-index: 1;
  transition: transform 0.3s ease;
}

.pointer:hover {
  transform: translateX(-50%) rotate(180deg) scale(1.1);
}

canvas {
  border-radius: 50%;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  transition: transform 0.1s linear, box-shadow 0.3s ease;
}

canvas:hover {
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.15);
}

.control-panel {
  display: flex;
  flex-direction: column;
  gap: 25px;
  width: 100%;
  max-width: 500px;
  padding: 20px;
  background: white;
  border-radius: 15px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
}

.speed-controls {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

.control-item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 10px;
  background: #f8f9fa;
  border-radius: 10px;
}

.control-item label {
  min-width: 100px;
  font-weight: 500;
  color: #2c3e50;
}

.control-item .slider {
  flex: 1;
  height: 10px;
  -webkit-appearance: none;
  background: linear-gradient(to right, #e0e0e0, #f5f5f5);
  border-radius: 5px;
  outline: none;
}

.control-item .slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  background: #2196F3;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  transition: transform 0.2s ease;
}

.control-item .slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
}

.control-item .value {
  min-width: 60px;
  text-align: center;
  font-weight: 500;
  color: #2c3e50;
}

.button-group {
  display: flex;
  gap: 15px;
  justify-content: center;
}

.control-panel button {
  padding: 12px 30px;
  font-size: 18px;
  font-weight: 500;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.edit-btn {
  background: linear-gradient(135deg, #2196F3, #1976D2);
  color: white;
}

.edit-btn:hover {
  background: linear-gradient(135deg, #1976D2, #1565C0);
  transform: translateY(-2px);
}

.start-btn {
  background: linear-gradient(135deg, #4CAF50, #45a049);
  color: white;
}

.start-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #45a049, #388E3C);
  transform: translateY(-2px);
}

.start-btn:disabled {
  background: #e0e0e0;
  cursor: not-allowed;
  transform: none;
}

.result {
  font-size: 28px;
  font-weight: 600;
  color: #1976D2;
  text-align: center;
  margin-top: 20px;
  padding: 20px 40px;
  background: linear-gradient(135deg, #ffffff, #f8f9fa);
  border-radius: 16px;
  box-shadow: 0 8px 16px rgba(25, 118, 210, 0.15);
  animation: fadeIn 0.5s ease;
  border: 2px solid rgba(25, 118, 210, 0.1);
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 弹窗样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
}

.settings-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  background: white;
  border: none;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  z-index: 10;
}

.settings-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.settings-btn svg {
  width: 100%;
  height: 100%;
  color: #666;
  transition: color 0.3s ease;
}

.settings-btn:hover svg {
  color: #333;
}

.settings-panel {
  max-width: 400px !important;
}

.settings-section {
  margin-bottom: 20px;
}

.settings-section h3 {
  margin: 0 0 15px 0;
  color: #333;
  font-size: 16px;
}

.setting-item {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
}

.setting-item label {
  min-width: 80px;
}

.setting-item input[type="color"] {
  width: 50px;
  height: 30px;
  padding: 0;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.modal-content {
  background: white;
  padding: 30px;
  border-radius: 16px;
  max-width: 600px;
  width: 90%;
  max-height: 85vh;
  overflow-y: auto;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
}

.modal-content h2 {
  margin: 0 0 20px 0;
  color: #333;
}

.batch-import {
  margin-bottom: 20px;
}

.batch-textarea {
  color: #666;
  font-size: 14px;
  margin-bottom: 10px;
}

.batch-textarea {
  width: 100%;
  height: 120px;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin: 10px 0;
  resize: vertical;
  font-size: 14px;
}

.modal-footer {
  text-align: right;
}

.close-btn {
  padding: 8px 16px;
  background-color: #757575;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.close-btn:hover {
  background-color: #616161;
}
.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.cancel-btn {
  padding: 8px 20px;
  background-color: #9e9e9e;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.cancel-btn:hover {
  background-color: #757575;
}

.save-btn {
  padding: 8px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.save-btn:hover {
  background-color: #45a049;
}
</style>