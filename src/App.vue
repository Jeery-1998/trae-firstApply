<script setup lang="ts">
import { ref } from 'vue';
import LuckyWheel from './components/LuckyWheel.vue';
import SettingsIcon from './components/SettingsIcon.vue';

const wheelSettings = ref({
  prizes: ['一等奖', '二等奖', '三等奖', '四等奖'],
  rotationSpeed: 10,
  rotationDuration: 5,
  title: '幸运转盘',
  wheelStyle: {
    pointerColor: '#e91e63',
    saturation: 70,
    lightness: 75
  }
});

const handleSettingsUpdate = (newSettings: any) => {
  console.log(newSettings,'nesSettings');
  wheelSettings.value = {
    prizes: [...newSettings.prizes],
    rotationSpeed: newSettings.rotationSpeed,
    rotationDuration: newSettings.rotationDuration,
    wheelStyle: {
      pointerColor: newSettings.wheelStyle.pointerColor,
      saturation: newSettings.wheelStyle.saturation,
      lightness: newSettings.wheelStyle.lightness
    },
    title: newSettings.title
  };
};
</script>

<template>
  <div class="app-container">
    <SettingsIcon
      :initialSettings="wheelSettings"
      @settings-updated="handleSettingsUpdate"
    />
    <h1>{{wheelSettings.title}}</h1>
    <LuckyWheel
      v-model:prizes="wheelSettings.prizes"
      v-model:rotationSpeed="wheelSettings.rotationSpeed"
      v-model:rotationDuration="wheelSettings.rotationDuration"
      v-model:wheelStyle="wheelSettings.wheelStyle"
      :wheelTitle="wheelSettings.title"
    />
  </div>
</template>

<style scoped>
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  color: #2c3e50;
  margin-bottom: 30px;
}
</style>
