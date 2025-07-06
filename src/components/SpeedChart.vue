<template>
  <canvas ref="chartCanvas"></canvas>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Chart } from 'chart.js/auto'

const chartCanvas = ref(null)

onMounted(() => {
  new Chart(chartCanvas.value, {
    type: 'line',
    data: {
      labels: ['12:00', '12:01', '12:02'],
      datasets: [{
        label: 'Speed (km/h)',
        data: [35, 38, 42],
        borderColor: 'red',
        tension: 0.3
      }]
    }
  })
})
</script>