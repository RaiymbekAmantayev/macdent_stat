<template>
  <div>
    <div class="chart-container">
      <canvas ref="chart1" width="850" height="300"></canvas>
    </div>
    <div class="chart-info">
      <!-- Здесь можете добавить любую информацию, которую хотите отображать -->
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
  props: {
    countCheck: Object,
    doctorTotal:Object,
    busy_Time: Object
  },
  mounted() {
    this.createChart('chart1', [], []);
  },
  watch: {
    countCheck(newCountCheck) {
      this.updateChart('chart1', Object.keys(newCountCheck), Object.values(newCountCheck));
    }
  },
  methods: {
    createChart(canvasId, labels, data) {
      const ctx = this.$refs[canvasId].getContext('2d');
      this.chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: '',
            data: data,
            backgroundColor: '#fff',
            borderColor: '#fff',
            borderWidth: 1
          }]
        },
        options: {
          indexAxis: 'y',
          scales: {
            yAxes: [{
              gridLines: {
                display: false,
                color: 'white'
              },
              ticks: {
                fontColor: 'white',
                beginAtZero: true
              }
            }],
            xAxes: [{
              gridLines: {
                display: false,
                color: 'white'
              },
              ticks: {
                fontColor: 'white'
              }
            }],
          },
          legend: {
            labels: {
              fontColor: 'white'
            }
          },
          barPercentage: 0.3,
          categoryPercentage: 0.5
        }
      });
    },
    updateChart(canvasId, labels, data) {
      if (this.chartInstance) {
        this.chartInstance.data.labels = labels;
        this.chartInstance.data.datasets[0].data = data;
        this.chartInstance.update();
      } else {
        this.createChart(canvasId, labels, data);
      }
    }
  }
};
</script>

<style scoped>
.chart-container {
  background-color: #fc3434;
  color: #fff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  width: 900px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
}

.chart-info {
  width: 900px;
  margin: 0 auto;
  text-align: center;
  padding: 20px 0;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  position: relative;
  height: 250px;
}

canvas {
  max-width: 1000px;
  max-height: 1000px;
}
</style>
