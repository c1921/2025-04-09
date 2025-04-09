<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { Bar } from 'vue-chartjs';
import { Chart as ChartJS, CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend } from 'chart.js';

// 注册Chart.js组件
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

const props = defineProps<{
  count: number
}>();

// 固定显示60个数据点
const MAX_DATA_POINTS = 60;

// 初始化图表数据 - 预填充60个空数据点，X轴从60到1倒序排列
const initialLabels = Array(MAX_DATA_POINTS).fill('').map((_, index) => `${MAX_DATA_POINTS - index}`);
const initialData = Array(MAX_DATA_POINTS).fill(0);

// 初始化图表数据
const chartData = ref({
  labels: initialLabels,
  datasets: [{
    label: 'Clicks Per Second',
    backgroundColor: 'rgba(66, 184, 131, 0.6)',
    borderColor: '#42b883',
    borderWidth: 1,
    data: initialData
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
let lastCount = 0;
let lastCountTime = Date.now();
let clicksThisSecond = 0;

// 处理数据更新，保持固定60个数据点
function updateChartData(clicksPerSecond: number) {
  const newData = [...chartData.value.datasets[0].data];
  newData.shift(); // 移除最旧的数据点
  newData.push(clicksPerSecond); // 添加新数据点
  
  // 更新图表数据，X轴标签保持不变
  chartData.value = {
    labels: chartData.value.labels,
    datasets: [{
      ...chartData.value.datasets[0],
      data: newData
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