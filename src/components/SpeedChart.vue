<template>
  <div class="chart-container">
    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, nextTick } from 'vue'
import { Chart } from 'chart.js/auto'

const props = defineProps({
  speed: {
    type: Number,
    default: 0
  },
  positionZ: {
    type: Number,
    default: 0
  },
  resetChart: {
    type: Boolean,
    default: false
  }
})

const chartCanvas = ref(null)
let chart = null

onMounted(() => {
  initChart()
})

const initChart = () => {
  if (chart) {
    chart.destroy()
  }
  
  chart = new Chart(chartCanvas.value, {
    type: 'scatter',
    data: {
      datasets: [{
        label: 'Speed (km/h)',
        data: [],
        borderColor: '#ff6b6b',
        backgroundColor: 'rgba(255, 107, 107, 0.1)',
        borderWidth: 3,
        tension: 0.3,
        fill: true,
        pointRadius: 2,
        pointHoverRadius: 6
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      animation: false,
      devicePixelRatio: window.devicePixelRatio || 1,
      interaction: {
        intersect: false,
        mode: 'index'
      },
      plugins: {
        legend: {
          display: true,
          position: 'top',
          labels: {
            font: {
              size: 14,
              weight: 'bold'
            }
          }
        }
      },
      scales: {
        x: {
          min: -500,
          max: 500,
          title: {
            display: true,
            text: 'Position Z',
            font: {
              size: 13,
              weight: 'bold'
            }
          },
          ticks: {
            stepSize: 100,
            font: {
              size: 12
            }
          }
        },
        y: {
          title: {
            display: true,
            text: 'Speed (km/h)',
            font: {
              size: 13,
              weight: 'bold'
            }
          },
          ticks: {
            font: {
              size: 12
            }
          },
          beginAtZero: true,
          max: 60
        }
      }
    }
  })
}

watch(
  () => [props.speed, props.positionZ, props.resetChart],
  ([newSpeed, newPositionZ, reset]) => {
    if (!chart) return

    if (reset) {
      // 차트 초기화
      chart.data.labels = []
      chart.data.datasets[0].data = []
      chart.update('none') // 애니메이션 없이 업데이트
      return
    }

    // 유효한 데이터가 있을 때만 추가
    if (typeof newSpeed === 'number' && typeof newPositionZ === 'number') {
      // 데이터 추가
        chart.data.datasets[0].data.push({
          x: Math.round(newPositionZ),
          y: newSpeed
        })
      
      // 최대 50개 데이터 포인트 유지
      if (chart.data.labels.length > 50) {
        chart.data.labels.shift()
        chart.data.datasets[0].data.shift()
      }
      
      chart.update('none') // 애니메이션 없이 업데이트
    }
  },
  { immediate: true, deep: true }
)
</script>

<style scoped>
.chart-container {
  width: 100%;
  height: 100%;
  position: relative;
  min-height: 240px;
  min-width: 360px;
}

canvas {
  width: 100% !important;
  height: 100% !important;
  display: block;
}
</style>