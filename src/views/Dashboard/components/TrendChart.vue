<template>
  <!-- 图表容器：使用flex填充父容器 -->
  <div class="chart-container">
    <div id="trendChart" class="echarts-container"></div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const props = defineProps({
  chartData: {
    type: Object,
    required: true,
    default: () => ({
      dates: [],
      customerData: [],
      leadData: [],
      contractData: [],
      approvedData: [],
      rejectedData: []
    })
  }
})

const chartInstance = ref<echarts.ECharts | null>(null)

// 初始化图表：添加审核趋势数据系列
const initChart = () => {
  const chartDom = document.getElementById('trendChart')
  if (!chartDom) return

  // 销毁旧实例
  if (chartInstance.value) chartInstance.value.dispose()

  // 初始化图表
  chartInstance.value = echarts.init(chartDom, null, {
    width: 'auto',
    height: 'auto',
    devicePixelRatio: window.devicePixelRatio || 1
  })

  // 检查数据是否存在，避免undefined错误
  const hasData = props.chartData && props.chartData.dates && props.chartData.dates.length > 0

  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' },
      textStyle: { fontSize: 14 }
    },
    legend: {
      data: ['新增客户', '新增线索', '新增合同', '审核通过', '审核拒绝'],
      top: 10,
      left: 'center',
      textStyle: { fontSize: 14 },
      // 处理图例过多的情况
      formatter: function (name: string) {
        return name.length > 4 ? name.substring(0, 4) + '...' : name
      }
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '15%',
      top: '15%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: hasData ? props.chartData.dates : [],
      position: 'bottom',
      axisLabel: {
        fontSize: 12,
        rotate: 45,
        margin: 15
      },
      axisLine: { show: true, lineStyle: { color: '#86909c' } }
    },
    yAxis: {
      type: 'value',
      min: 0,
      axisLabel: { fontSize: 12 },
      axisLine: { show: true, lineStyle: { color: '#86909c' } },
      splitLine: { lineStyle: { color: '#eee' } }
    },
    series: [
      {
        name: '新增客户',
        type: 'bar', // 明确指定类型，修复原错误
        data: hasData ? props.chartData.customerData : [],
        itemStyle: { color: '#4285f4' },
        barWidth: '18%'
      },
      {
        name: '新增线索',
        type: 'bar', // 明确指定类型
        data: hasData ? props.chartData.leadData : [],
        itemStyle: { color: '#00b42a' },
        barWidth: '18%'
      },
      {
        name: '新增合同',
        type: 'bar', // 明确指定类型
        data: hasData ? props.chartData.contractData : [],
        itemStyle: { color: '#ff7d00' },
        barWidth: '18%'
      },
      // 新增审核通过趋势
      {
        name: '审核通过',
        type: 'bar', // 明确指定类型
        data: hasData ? props.chartData.approvedData : [],
        itemStyle: { color: '#27ae60' },
        barWidth: '18%'
      },
      // 新增审核拒绝趋势
      {
        name: '审核拒绝',
        type: 'bar', // 明确指定类型
        data: hasData ? props.chartData.rejectedData : [],
        itemStyle: { color: '#e74c3c' },
        barWidth: '18%'
      }
    ]
  }

  chartInstance.value.setOption(option)
}

// 监听窗口大小变化
const handleResize = () => {
  if (chartInstance.value) {
    chartInstance.value.resize()
  }
}

// 监听数据变化
watch(() => props.chartData, initChart, { deep: true })

// 初始化
onMounted(() => {
  initChart()
  window.addEventListener('resize', handleResize)
})

// 清理
onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (chartInstance.value) {
    chartInstance.value.dispose()
  }
})
</script>

<style scoped>
.chart-container {
  width: 100%;
  height: 100%;
  display: flex;
}

.echarts-container {
  width: 100%;
  height: 100%;
  min-height: 400px; /* 确保图表有足够高度 */
}
</style>
