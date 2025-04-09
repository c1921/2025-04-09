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
    label: '点击总数',
    backgroundColor: 'rgba(25, 118, 210, 0.6)',
    borderColor: '#1976d2',
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
        text: '秒前'
      },
      // 反转X轴，使数据从右向左更新，但显示的标签仍然是60到1
      reverse: true
    },
    y: {
      beginAtZero: true
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

// 处理数据更新，保持固定60个数据点
function updateChartData(newValue: number) {
  const newData = [...chartData.value.datasets[0].data];
  newData.shift(); // 移除最旧的数据点
  newData.push(newValue); // 添加新数据点
  
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
  // 每秒更新一次图表数据
  const interval = setInterval(() => {
    // 直接使用总点击数
    updateChartData(props.count);
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