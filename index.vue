
<template>
  <div class="container p-4 flex flex-col items-center">
    <!-- 查询城市按钮 -->
    <button @click="showModal = true" class="bg-blue-500 text-white px-6 py-3 rounded-lg shadow-lg hover:bg-blue-600 transition-all">
      查询城市
    </button>

    <!-- 弹出窗口（带动画） -->
    <transition name="modal-fade">
      <div v-if="showModal" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50">
        <div class="bg-white p-6 rounded-lg shadow-2xl w-96 transform transition-all scale-95 opacity-0 modal-enter">
          <h2 class="text-lg font-semibold mb-4 text-center">输入城市名称</h2>
          <input v-model="city" @input="fetchCoordinates" placeholder="输入城市" class="border p-3 rounded-lg w-full focus:ring-2 focus:ring-blue-400 transition" />
          <div v-if="latitude && longitude" class="mt-4 text-sm text-gray-600 text-center">
            <p>🌍 纬度: {{ latitude }}</p>
            <p>🌎 经度: {{ longitude }}</p>
          </div>
          <div class="flex justify-between mt-6">
            <button @click="showModal = false" class="px-4 py-2 bg-gray-300 rounded-lg hover:bg-gray-400 transition">取消</button>
            <button @click="confirmCity" class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition">确认</button>
          </div>
        </div>
      </div>

    </transition>

    <!-- 天气数据 & 可视化图表 -->
    <div v-if="weatherData" class="mt-6 w-full max-w-lg text-center">
      <h2 class="text-2xl font-semibold text-gray-800">🌤 天气信息（{{ city }}）</h2>
      <p class="text-lg text-gray-700 mt-2">当前温度：{{ weatherData.current.temperature_2m }}°C</p>

      <div ref="chartEl" style="width: 100%; height: 400px;"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue';
import * as echarts from 'echarts';
import axios from 'axios';

const showModal = ref(false);
const city = ref('');
const latitude = ref(null);
const longitude = ref(null);
const weatherData = ref(null);
const chartEl = ref(null);

async function fetchCoordinates() {
  if (!city.value) return;
  const apiKey = 'a5b6fe84acda49a99de7eb6d485d906d';
  const url = `https://api.opencagedata.com/geocode/v1/json?q=${encodeURIComponent(city.value)}&key=${apiKey}`;
  try {
    const response = await fetch(url);
    const data = await response.json();
    if (data.results.length > 0) {
      latitude.value = data.results[0].geometry.lat;
      longitude.value = data.results[0].geometry.lng;
    }
  } catch (error) {
    console.error('获取经纬度失败:', error);
  }
}

async function confirmCity() {
  if (!latitude.value || !longitude.value) return alert('加载中！');
  showModal.value = false;
  await fetchWeatherData();
}

async function fetchWeatherData() {
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${latitude.value}&longitude=${longitude.value}&current=temperature_2m&daily=temperature_2m_max,temperature_2m_min&timezone=auto`;
  try {
    const response = await axios.get(url);
    weatherData.value = response.data;
    nextTick(() => drawChart());
  } catch (error) {
    console.error('获取天气数据失败:', error);
  }
}

function drawChart() {
  if (!weatherData.value) return;
  const chart = echarts.init(chartEl.value);
  const option = {
    title: {text: '过去7天温度变化', left: 'center'},
    tooltip: {trigger: 'axis'},
    xAxis: {type: 'category', data: weatherData.value.daily.time},
    yAxis: {type: 'value'},
    series: [
      {name: '最高温', type: 'line', data: weatherData.value.daily.temperature_2m_max},
      {name: '最低温', type: 'line', data: weatherData.value.daily.temperature_2m_min}
    ]
  };
  chart.setOption(option);
  window.addEventListener('resize', () => chart.resize());
}
</script>

<style>
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.3s, transform 0.3s;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.modal-enter {
  animation: modal-zoom 0.3s ease-out forwards;
}

@keyframes modal-zoom {
  from {
    transform: scale(0.9);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}
</style>
