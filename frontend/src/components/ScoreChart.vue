<template>
  <v-card class="score-chart" elevation="2">
    <v-card-title class="d-flex align-center title-section">
      <v-icon icon="mdi-chart-line" class="mr-2 title-icon"></v-icon>
      评分趋势
      <v-spacer></v-spacer>
      <v-btn
        icon="mdi-refresh"
        variant="text"
        size="small"
        :loading="loading"
        @click="refreshData"
        class="refresh-btn"
      ></v-btn>
    </v-card-title>

    <v-card-text class="chart-section">
      <div ref="chartRef" class="chart-container"></div>
    </v-card-text>
  </v-card>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'

// 定义 ECharts 提示框参数类型
interface TooltipParams {
  name: string
  value: number
  seriesName: string
  seriesType: string
  dataIndex: number
}

const loading = ref(false)
const chartRef = ref<HTMLElement | null>(null)
let chart: echarts.ECharts | null = null

// 模拟数据
const mockData = {
  dates: ['2024-03-18', '2024-03-19', '2024-03-20', '2024-03-21', '2024-03-22', '2024-03-23', '2024-03-24'],
  scores: [85, 88, 92, 90, 95, 89, 93]
}

// 初始化图表
const initChart = () => {
  if (!chartRef.value) return

  chart = echarts.init(chartRef.value)
  const option = {
    tooltip: {
      trigger: 'axis',
      formatter: function(params: TooltipParams[]) {
        const data = params[0]
        return `${data.name}<br/>评分：${data.value}分`
      },
      backgroundColor: 'rgba(0, 0, 0, 0.8)',
      borderColor: 'transparent',
      textStyle: {
        color: '#fff'
      }
    },
    grid: {
      top: '10%',
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: mockData.dates,
      axisLine: {
        lineStyle: {
          color: '#666'
        }
      },
      axisLabel: {
        color: '#666',
        formatter: (value: string) => value.substring(5) // 只显示月-日
      }
    },
    yAxis: {
      type: 'value',
      min: 0,
      max: 100,
      interval: 20,
      axisLine: {
        show: true,
        lineStyle: {
          color: '#666'
        }
      },
      axisLabel: {
        color: '#666'
      },
      splitLine: {
        lineStyle: {
          color: '#eee'
        }
      }
    },
    series: [
      {
        name: '评分',
        type: 'line',
        data: mockData.scores,
        smooth: true,
        symbol: 'circle',
        symbolSize: 8,
        itemStyle: {
          color: '#1976d2'
        },
        lineStyle: {
          width: 3,
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: '#1976d2' },
            { offset: 1, color: '#2196f3' }
          ])
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: 'rgba(25, 118, 210, 0.3)' },
            { offset: 1, color: 'rgba(33, 150, 243, 0.1)' }
          ])
        }
      }
    ]
  }

  chart.setOption(option)
}

// 刷新数据
const refreshData = async () => {
  loading.value = true
  try {
    // 模拟API调用
    await new Promise(resolve => setTimeout(resolve, 1000))
    // 更新数据
    mockData.scores = mockData.scores.map(() => Math.floor(Math.random() * 20) + 80)
    if (chart) {
      chart.setOption({
        series: [{
          data: mockData.scores
        }]
      })
    }
  } catch (error) {
    console.error('刷新数据失败:', error)
  } finally {
    loading.value = false
  }
}

// 监听窗口大小变化
const handleResize = () => {
  chart?.resize()
}

onMounted(() => {
  initChart()
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  chart?.dispose()
})
</script>

<style scoped>
.score-chart {
  height: 100%;
  border-radius: 12px;
  background: linear-gradient(to bottom right, #ffffff, #f8f9fa);
  display: flex;
  flex-direction: column;
}

.title-section {
  background: linear-gradient(45deg, #1976d2, #2196f3);
  color: white;
  padding: 16px 24px;
  border-radius: 12px 12px 0 0;
}

.title-icon {
  color: white;
}

.refresh-btn {
  color: white;
  opacity: 0.8;
  transition: opacity 0.2s;
}

.refresh-btn:hover {
  opacity: 1;
}

.chart-section {
  flex: 1;
  padding: 16px;
  position: relative;
}

.chart-container {
  width: 100%;
  height: 100%;
  min-height: 300px;
}
</style> 