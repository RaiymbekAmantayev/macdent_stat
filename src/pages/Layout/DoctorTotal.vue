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
    <div class="chart-infoFirst">
      Количество чеков по врачам
      <!-- Здесь можете добавить любую информацию, которую хотите отображать -->
    </div>
    <div class="chart-infoSecond">
      Тотал кассы по врачам
      <!-- Здесь можете добавить любую информацию, которую хотите отображать -->
    </div>
    <div class="chart-infoThird">
      Количество часов принимавших пациентов
      <!-- Здесь можете добавить любую информацию, которую хотите отображать -->
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
    this.createChart('chart1', [], []);
    this.createChart('chart2', [], []);
    this.createChart('chart3', [], []);
  },
  watch: {
    countCheck(newCountCheck) {
      this.updateChart('chart1', Object.keys(newCountCheck), Object.values(newCountCheck),'Количество чеков по врачам' );
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
.chart-containerFirst {
  background-color: #fc3434;
  margin-top: 70px;
  color: #fff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  width: 900px;
  position: relative;
  z-index: 1;
  margin-left: 180px;
  display: flex;
  justify-content: space-between;
}

.chart-containerSecond {
  background-color: #0066FF;
  margin-top: 70px;
  color: #fff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  width: 900px;
  position: relative;
  z-index: 1;
  margin-left: 180px;
  display: flex;
  justify-content: space-between;
}

.chart-containerThird {
  background-color: #009900;
  margin-top: 70px;
  color: #fff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  width: 900px;
  position: relative;
  z-index: 1;
  margin-left: 180px;
  display: flex;
  justify-content: space-between;
}

.chart-infoFirst {
  width: 900px;
  margin: 0 auto;
  text-align: center;
  padding: 20px 0;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  position: absolute;
  right: 223px;
  top: 2009px;
  height: 320px;
}

.chart-infoSecond {
  width: 900px;
  margin: 0 auto;
  text-align: center;
  padding: 20px 0;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  position: absolute;
  right: 223px;
  top: 2409px;
  height: 320px;
}

.chart-infoThird {
  width: 900px;
  margin: 0 auto;
  text-align: center;
  padding: 20px 0;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  position: absolute;
  right: 223px;
  top: 1609px;
  height: 320px;
}

canvas {
  max-width: 1000px;
  max-height: 1000px;
}
</style>
