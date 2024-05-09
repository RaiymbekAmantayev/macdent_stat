<template>
  <div style="margin-top: 5%" class="chartWrapper">
    <div class="btn-container">
      <button @click="updateChart('hour')">День</button>
      <button @click="updateChart('day')">Неделя</button>
      <button @click="updateChart('weekday')">Месяц</button>
    </div>

    <div id="chart-container">
      <canvas id="myChart-Records"></canvas>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';
import axios from "axios";
import {API_KEY} from "@/pages/API_KEY";

export default {
  data() {
    return {
      NewClient : [],
      OldClient: [],
      label: [],
      times:["09:00", "10:00", "11:00", "12:00", "13:00", "14:00", "15:00","16:00","17:00","18:00", "19:00","20:00"],
      initialData: {
        labels: this.label,
        datasets: [
          {
            label: 'Первый раз',
            data: this.NewClient,
            backgroundColor: '#31b7e3',
          },
          {
            label: 'Старые',
            data: this.OldClient,
            backgroundColor: '#070551',
          }
        ]
      },
      initialDay: {
        labels: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
        datasets: [
          {
            label: 'Первый раз',
            data: [400, 500, 90, 151, 206, 155, 400],
            backgroundColor: '#31b7e3',
          },
          {
            label: 'Старые',
            data: [65, 400, 350, 220, 100, 600, 400],
            backgroundColor: '#070551',
          }
        ]
      },
      initialMonth: {
        labels: ['Янв', 'Фев', 'Мар', 'Апр', 'Май', 'Июн', 'Июл'],
        datasets: [
          {
            label: 'Первый раз',
            data: [100, 55, 70, 180, 200, 160, 300],
            backgroundColor: '#31b7e3',
          },
          {
            label: 'Старые',
            data: [405, 100, 310, 200, 160, 300, 410],
            backgroundColor: '#070551',
          }
        ]
      },
      myChart: null,
      allRecordsByPeriod: null
    };
  },
  async mounted() {
    const today = new Date()
    const StartMonth = new Date(today.getFullYear(), today.getMonth() - 1, today.getDate())
    const endMonth = new Date(today.getFullYear(), today.getMonth(), today.getDate())
    const startDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
    const endDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());

    try{
      let recordsPromises = []; // Используем другое имя переменной
      let currentPage = 1;
      let hasRecords = true;
      while (hasRecords) {
        const response = await axios.get(`https://api-developer.macdent.kz/zapis/find/?page=${currentPage}&access_token=${API_KEY}`);
        if (response.data.zapisi.length > 0) {
          recordsPromises = recordsPromises.concat(response.data.zapisi); // Обновляем массив
          currentPage++;
        } else {
          hasRecords = false;
        }
      }
      const Records = await Promise.all(recordsPromises);
      this.allRecordsByPeriod = this.filterPaymentsByDateRange(Records, StartMonth, endMonth)
      var newData = {};
      let values = this.getTotalRecod(this.allRecordsByPeriod , startDay, endDay);
      this.label = Object.keys(this.result);
      let newValue = values.map(arr => arr[0]);
      let oldValue = values.map(arr => arr[1]);
      this.NewClient = newValue
      this.OldClient = oldValue
      newData = {
        labels: this.label,
        datasets: [
          {
            label: 'Первый раз',
            data: this.NewClient,
            backgroundColor: '#31b7e3',
          },
          {
            label: 'Старые',
            data: this.OldClient,
            backgroundColor: '#070551',
          }
        ]
      };

      // Установка данных в initialData
      this.initialData.labels = newData.labels;
      this.initialData.datasets = newData.datasets;

      // Вызов метода updateChart() после установки initialData
      this.updateChart();
    }catch (e){
      // console.log(e)
    }
    this.initChart();
  },
  watch:{
    initialData(newData){
      // console.log("labels: ",newData.labels)
    },
    myChart(newChart){
      // console.log("chartNew: ",newChart.data)
    },
    NewClient(newClient) {
      // console.log("clientNew: ",newClient)
    },
    OldClient(OldClient) {
      // console.log("clientOld: ",OldClient)
    },
  },
  methods: {
    filterPaymentsByDateRange(ResultQuery, startDate, endDate) {
      return ResultQuery.filter(payment => {
        const [day, month, year] = payment.date.split('.');
        const paymentDate = new Date(year, month - 1, day);
        return paymentDate >= startDate && paymentDate <= endDate;
      });
    },
    initChart() {
      const ctx = document.getElementById('myChart-Records').getContext('2d');

      // Уничтожаем предыдущий график, если он существует
      if (this.myChart) {
        this.myChart.destroy();
      }

      this.myChart = new Chart(ctx, {
        type: 'bar',
        data: this.initialData,
        options: {
          animation: {
            duration: 3000
          },
          maintainAspectRatio: false,
          scales: {
            y: {
              stacked: true,
              beginAtZero: true
            },
            x: {
              stacked: true
            }
          },
          plugins: {
            legend: {
              display: false
            }
          },
          barPercentage: 0.3
        }
      });
    },
    getTotalRecod(allRecords, startDate, endDate) {
      let allRecord = this.filterPaymentsByDateRange(allRecords, startDate, endDate)
      let recordsByHour = {}; // Изменено: теперь это объект, где каждый час имеет свой собственный массив
      allRecord.forEach(record => {
        let startTime = record.start;
        let startHour = startTime.split(' ')[1].split(':')[0];
        startHour = parseInt(startHour);
        let endTime = record.end;
        let endHour = endTime.split(' ')[1].split(':')[0];

        startHour = parseInt(startHour);
        endHour = parseInt(endHour);

        if (startHour === endHour) {
          const hourString = startHour < 10 ? "0" + startHour + ":00" : startHour + ":00";
          if (this.times.includes(hourString)) {
            // Изменено: проверяем, есть ли массив для этого часа, и создаем его, если он не существует
            if (!recordsByHour[hourString]) {
              recordsByHour[hourString] = [0, 0];
            }
            // Изменено: добавляем запись в соответствующий массив, основываясь на значении record.isFirst
            const index = record.isFirst ? 0 : 1;
            recordsByHour[hourString][index]++; // Инкрементируем соответствующий индекс массива
          }
        } else {
          endHour -= 1;
          for (let hour = startHour; hour <= endHour; hour++) {
            const hourString = hour < 10 ? "0" + hour + ":00" : hour + ":00";
            if (this.times.includes(hourString)) {
              // Изменено: проверяем, есть ли массив для этого часа, и создаем его, если он не существует
              if (!recordsByHour[hourString]) {
                recordsByHour[hourString] = [0, 0];
              }
              // Изменено: добавляем запись в соответствующий массив, основываясь на значении record.isFirst
              const index = record.isFirst ? 0 : 1;
              recordsByHour[hourString][index]++; // Инкрементируем соответствующий индекс массива
            }
          }
        }
      });
      const sortedKeys = Object.keys(recordsByHour)
          .sort((a, b) => {
            const timeA = parseInt(a.split(':')[0]);
            const timeB = parseInt(b.split(':')[0]);
            return timeA - timeB;
          });


      const sortedResult = {};
      sortedKeys.forEach(key => {
        sortedResult[key] =recordsByHour[key];
      });

      this.result = sortedResult;
      let values = Object.values(this.result)
      let label = Object.keys(this.result)
      // console.log("values: ",values.map(arr => arr[0]))
      let newValue = values.map(arr => arr[0]);
      let oldValue = values.map(arr => arr[1]);
      this.label = label
      this.NewClient = newValue
      this.OldClient = oldValue
      return values
    },
    updateChart(interval) {
      const today = new Date();
      var newData = {};
      switch (interval) {
        case 'hour':
          const startDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
          const endDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
          this.getTotalRecod(this.allRecordsByPeriod, startDay, endDay);
          newData = {
            labels: this.label,
            datasets: [
              {
                label: 'Первый раз',
                data: this.NewClient,
                backgroundColor: '#31b7e3',
              },
              {
                label: 'Старые',
                data: this.OldClient,
                backgroundColor: '#070551',
              }
            ]
          };
          break;
        case 'day':
          const startWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate() - 7);
          const endWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate());
          this.getTotalRecod(this.allRecordsByPeriod, startWeek, endWeek);
          newData = {
            labels: this.label,
            datasets: [
              {
                label: 'Первый раз',
                data: this.NewClient,
                backgroundColor: '#31b7e3',
              },
              {
                label: 'Старые',
                data: this.OldClient,
                backgroundColor: '#070551',
              }
            ]
          };
          break;
        case 'weekday':
          const StartMonth = new Date(today.getFullYear(), today.getMonth() - 1, today.getDate())
          const endMonth = new Date(today.getFullYear(), today.getMonth(), today.getDate())
          this.getTotalRecod(this.allRecordsByPeriod, StartMonth, endMonth);
          newData = {
            labels: this.label,
            datasets: [
              {
                label: 'Первый раз',
                data: this.NewClient,
                backgroundColor: '#31b7e3',
              },
              {
                label: 'Старые',
                data: this.OldClient,
                backgroundColor: '#070551',
              }
            ]
          };
          break;
      }
      this.myChart.data = newData;
      this.myChart.update();
    }
  }
};
</script>


<style>
#chart-container {
  max-width: 900px;
  box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.1);
  margin: 0 auto;
  padding: 20px;
  margin-bottom: 15px;
  height: 300px;
}

#myChart-Records {
  height: 350px;
}

.btn-container {
  max-width: 900px;
  margin: 0 auto;
  margin-bottom: 20px;
  display: flex;
  justify-content: flex-end;
}

.btn-container button {
  padding: 10px 20px;
  margin-right: 10px;
  font-size: 16px;
  background-color: #030352;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

.btn-container button:hover {
  background-color: #020235;
}

.btn-container button:focus {
  outline: none;
}
</style>
