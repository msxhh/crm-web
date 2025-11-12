<template>
  <div id="tradeBar" style="height: 100%; width: 100%"></div>
</template>

<script lang="ts" name="CustomerDataChart" setup>
import * as echarts from 'echarts'
import { onMounted, ref, onUnmounted, PropType, watch } from 'vue'
import { TradeArray } from '@/api/interface'

// 定义组件接收的props，确保传入符合格式的图表数据
const props = defineProps({
  data: {
    type: Object as PropType<TradeArray>,
    required: true
  }
})

// 存储echarts图表实例
const myChart = ref<echarts.ECharts | null>(null)

// 初始化图表的核心方法
const initChart = () => {
  // 先判断数据中的时间列表是否有内容，避免空数据渲染
  if (props.data.timeList.length > 0) {
    const chartDom = document.getElementById('tradeBar')
    if (chartDom) {
      // 若已有图表实例，先销毁避免重复创建
      if (myChart.value) {
        myChart.value.dispose()
      }
      // 初始化新的图表实例
      myChart.value = echarts.init(chartDom)

      // 图表配置项
      const option = {
        tooltip: {
          trigger: 'item',
          axisPointer: {
            type: 'cross',
            crossStyle: {
              color: '#999'
            }
          }
        },
        legend: {
          data: ['客户数量']
        },
        xAxis: [
          {
            type: 'category',
            data: props.data.timeList,
            axisPointer: {
              type: 'shadow'
            }
          }
        ],
        yAxis: [
          {
            type: 'value',
            name: '客户数量'
          }
        ],
        series: [
          {
            name: '客户总量',
            type: 'bar',
            data: props.data.countList
          },
          {
            name: '买入交易总量',
            type: 'line',
            data: props.data.countList
          }
        ]
      }

      // 渲染图表
      myChart.value.setOption(option)

      // 窗口大小变化时，图表自适应调整
      const handleResize = () => {
        myChart.value?.resize()
      }
      window.addEventListener('resize', handleResize)

      // 组件卸载时清理资源，防止内存泄漏
      onUnmounted(() => {
        window.removeEventListener('resize', handleResize)
        myChart.value?.dispose()
      })
    }
  }
}

// 监听props数据变化，数据更新时重新渲染图表
watch(
  () => props.data,
  (newData) => {
    if (newData.timeList.length > 0) {
      initChart()
    }
  },
  { immediate: true } // 组件初始化时立即执行一次监听回调
)

// DOM挂载完成后，检查数据并初始化图表
onMounted(() => {
  if (props.data.timeList && props.data.timeList.length > 0) {
    initChart()
  }
})
</script>
