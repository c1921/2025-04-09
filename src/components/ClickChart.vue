<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { Bar } from 'vue-chartjs';
import { Chart as ChartJS, CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend } from 'chart.js';

// 注册Chart.js组件
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

const props = defineProps<{
  count: number
}>();

// 固定显示60个数据点，但存储最多3600个数据点
const DISPLAY_DATA_POINTS = 60;
const MAX_STORED_DATA_POINTS = 3600;

// 从localStorage加载数据的函数
function loadFromStorage(key: string, defaultValue: any) {
  try {
    const storedValue = localStorage.getItem(key)
    return storedValue ? JSON.parse(storedValue) : defaultValue
  } catch (error) {
    console.error(`Error loading ${key} from localStorage:`, error)
    return defaultValue
  }
}

// 保存数据到localStorage的函数
function saveToStorage(key: string, value: any) {
  try {
    localStorage.setItem(key, JSON.stringify(value))
  } catch (error) {
    console.error(`Error saving ${key} to localStorage:`, error)
  }
}

// 加载历史数据或创建新的数组
const storedClicksData = loadFromStorage('clicksPerSecondData', []);

// 确保数据长度不超过最大值
const clicksData = storedClicksData.slice(-MAX_STORED_DATA_POINTS);

// 创建X轴标签 - 使用相对位置而不是真实时间
const generateSequentialLabels = (count: number) => {
  return Array(count).fill(0).map((_, index) => (count - index).toString());
};

// 初始化图表数据
const chartData = ref({
  labels: generateSequentialLabels(DISPLAY_DATA_POINTS),
  datasets: [{
    label: 'Clicks Per Second',
    backgroundColor: 'rgba(66, 184, 131, 0.6)',
    borderColor: '#42b883',
    borderWidth: 1,
    data: clicksData.length >= DISPLAY_DATA_POINTS 
      ? clicksData.slice(-DISPLAY_DATA_POINTS) 
      : [...Array(DISPLAY_DATA_POINTS - clicksData.length).fill(0), ...clicksData]
  }]
});

// 图表配置
const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    x: {
      title: {
        display: true,
        text: 'Seconds Ago'
      },
      // 反转X轴，使数据从右向左更新，但显示的标签仍然是60到1
      reverse: true
    },
    y: {
      beginAtZero: true,
      suggestedMin: 0,
      suggestedMax: 5
    }
  },
  animation: {
    duration: 0 // 设置动画持续时间为0，相当于禁用动画
  },
  transitions: {
    active: {
      animation: {
        duration: 0 // 禁用过渡动画
      }
    }
  }
};

// 上一秒的计数和时间戳
let lastCount = props.count;
let lastCountTime = Date.now();
let clicksThisSecond = 0;

// 处理数据更新，只更新数据不更新时间标签
function updateChartData(clicksPerSecond: number) {
  // 添加新数据到历史记录
  clicksData.push(clicksPerSecond);
  
  // 如果数据超出最大存储限制，移除最旧的数据
  if (clicksData.length > MAX_STORED_DATA_POINTS) {
    clicksData.shift();
  }
  
  // 保存到localStorage
  saveToStorage('clicksPerSecondData', clicksData);
  
  // 更新图表显示 - 只显示最近的60个数据点
  const displayData = clicksData.slice(-DISPLAY_DATA_POINTS);
  
  // 如果数据少于60个，用0填充左侧
  const paddedData = displayData.length < DISPLAY_DATA_POINTS
    ? [...Array(DISPLAY_DATA_POINTS - displayData.length).fill(0), ...displayData]
    : displayData;
  
  // 更新图表数据，保持标签不变
  chartData.value = {
    labels: generateSequentialLabels(DISPLAY_DATA_POINTS),
    datasets: [{
      ...chartData.value.datasets[0],
      data: paddedData
    }]
  };
}

// 初始化计时器和数据更新
onMounted(() => {
  // 设置初始计数
  lastCount = props.count;
  lastCountTime = Date.now();
  
  // 每秒更新一次图表数据
  const interval = setInterval(() => {
    const now = Date.now();
    // 如果至少过了900毫秒（接近1秒）
    if (now - lastCountTime >= 900) {
      // 计算每秒的点击次数
      clicksThisSecond = props.count - lastCount;
      // 确保不为负数
      clicksThisSecond = Math.max(0, clicksThisSecond);
      
      // 更新图表数据
      updateChartData(clicksThisSecond);
      
      // 重置计数和时间戳
      lastCount = props.count;
      lastCountTime = now;
    }
  }, 1000);
  
  // 组件卸载时清除定时器
  onUnmounted(() => {
    clearInterval(interval);
  });
});
</script>

<template>
  <div class="chart-container">
    <Bar :data="chartData" :options="chartOptions" height="250" />
  </div>
</template>

<style scoped>
.chart-container {
  margin-top: 2rem;
  position: relative;
  height: 250px;
}
</style> 