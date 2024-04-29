<template>
  <div>
    <div class="chart-containerFirst">
      <canvas ref="chart1" width="850" height="300"></canvas>
    </div>
    <div class="chart-containerSecond">
      <canvas ref="chart2" width="850" height="300"></canvas>
    </div>
    <div class="chart-containerThird">
      <canvas ref="chart3" width="850" height="300"></canvas>
    </div>
    <div class="chart-info-wrapper">
      <div class="chart-infoFirst">
        <!-- Количество чеков по врачам -->
        <!-- Здесь можете добавить любую информацию, которую хотите отображать -->
      </div>
      <div class="chart-infoSecond">
        <!-- Тотал кассы по врачам -->
        <!-- Здесь можете добавить любую информацию, которую хотите отображать -->
      </div>
      <div class="chart-infoThird">
        <!-- Количество часов принимавших пациентов -->
        <!-- Здесь можете добавить любую информацию, которую хотите отображать -->
      </div>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
  props: {
    countCheck: Object,
    doctorTotal: Object,
    busy_time: Object
  },
  data() {
    return {
      chartInstances: {} // Хранение экземпляров графиков
    };
  },
  mounted() {
    this.createChart('chart1', [], [], 'Количество чеков по врачам');
    this.createChart('chart2', [], [], 'Тотал кассы по врачам');
    this.createChart('chart3', [], [], 'Количество часов принимавших пациентов');
  },
  watch: {
    countCheck(newCountCheck) {
      this.updateChart('chart1', Object.keys(newCountCheck), Object.values(newCountCheck), 'Количество чеков по врачам');
      // console.log("count: ",newCountCheck)
    },
    doctorTotal(newDoctorTotal) {
      this.updateChart('chart2', Object.keys(newDoctorTotal), Object.values(newDoctorTotal), 'Тотал кассы по врачам');
      // console.log("total: ", newDoctorTotal)
    },
    busy_time(newBusyTime) {
      this.updateChart('chart3', Object.keys(newBusyTime), Object.values(newBusyTime), 'Количество часов принимавших пациентов');
      // console.log("busy time: ", newBusyTime)
    }
  },
  methods: {
    createChart(canvasId, labels, data, label) {
      const ctx = this.$refs[canvasId].getContext('2d');
      const chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: label,
            data: data,
            backgroundColor: '#fff',
            borderColor: '#fff',
            borderWidth: 1
          }]
        },
        options: {
          responsive: false,
          // maintainAspectRatio: false,
          indexAxis: 'y',
          scales: {
            y: {
              grid: {
                color: 'rgba(255, 255, 255, 0.3)'
              },
              ticks: {
                color: 'white',
                beginAtZero: true,
                font: {
                  size: 15,
                  family: 'Helvetica',
                }
              }
            },

            x: {
              grid: {
                color: 'rgba(255, 255, 255, 0.3)'
              },
              ticks: {
                color: 'white'
              }
            },
          },
          plugins: {
            legend: {
              display: false
            }
          },
          barPercentage: 0.6,
          categoryPercentage: 0.5
        }
      });
      this.chartInstances[canvasId] = chartInstance; // Сохранение экземпляра графика
    },
    updateChart(canvasId, labels, data) {
      const chartInstance = this.chartInstances[canvasId];
      if (chartInstance) {
        chartInstance.data.labels = labels;
        chartInstance.data.datasets[0].data = data;
        chartInstance.update();
      } else {
        this.createChart(canvasId, labels, data);
      }
    }
  }
};

</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

.chart-containerFirst {
  background-color: #FC6E51;
  margin-top: 70px;
  color: #fff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0px 0px 20px 0px #ff5432;
  width: 900px;
  position: relative;
  z-index: 1;
  margin-left: 180px;
  display: flex;
  justify-content: space-between;
  height: 300;
}

.chart-containerSecond {
  background-color: #7fabed;
  margin-top: 70px;
  color: #fff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0px 0px 20px 0px #78aeff;
  width: 900px;
  position: relative;
  z-index: 1;
  margin-left: 180px;
  display: flex;
  justify-content: space-between;
  height: 300;
}

.chart-containerThird {
  background-color: #73c173;
  margin-top: 70px;
  color: #fff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0px 0px 20px 0px #54c954;
  width: 900px;
  position: relative;
  z-index: 1;
  margin-left: 180px;
  display: flex;
  justify-content: space-between;
  height: 300;
}

.chart-info-wrapper {
  display: flex;
}

.chart-infoFirst {
  width: 900px;
  margin: 0 auto;
  text-align: center;
  opacity: 0.7;
  padding: 20px 0;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  position: absolute;
  right: 209px;
  top: 2186px;
  height: 320px;
}

.chart-infoSecond {
  width: 900px;
  margin: 0 auto;
  text-align: center;
  padding: 20px 0;
  opacity: 0.7;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  position: absolute;
  right: 209px;
  top: 2586px;
  height: 320px;
}

.chart-infoThird {
  width: 900px;
  margin: 0 auto;
  text-align: center;
  padding: 20px 0;
  opacity: 0.7;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  position: absolute;
  right: 209px;
  top: 1767px;
  height: 320px;
}

canvas {
  max-width: 1000px;
  max-height: 1000px;
}

@media screen and (max-width: 800px) {
  canvas {
    max-width: 97%;
    max-height: 137px;
  }

  .chart-containerFirst {
    max-width: 97%;
    margin-bottom: 15px;
    margin-left: 0;
    padding: 0;
    margin: 0 auto 15px;
    height: none;
  }

  .chart-containerSecond {
    max-width: 97%;
    margin-bottom: 15px;
    padding: 0;
    margin-left: 0;
    margin: 0 auto 15px;
  }

  .chart-containerThird {
    max-width: 97%;
    margin-bottom: 15px;
    padding: 0;
    margin: 0 auto 15px;
  }

  .chart-infoFirst {
    display: none;
    width: 80px;
    padding: 0 0;
    margin: 0;
  }

  .chart-infoSecond {
    display: none;
    width: 80px;
    padding: 0 0;
    margin: 0;
  }


  .chart-infoThird {
    display: none;
    width: 80px;
    padding: 0 0;
    margin: 0;
  }
}
</style>
