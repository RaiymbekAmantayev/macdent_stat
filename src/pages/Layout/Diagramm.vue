<template>
  <div style="margin-left: 2%">
    <p>Show by period</p>
    <div class="text">
    <h1 >Аналитика данных</h1>
      <p>период: 2023-2024</p>
      <select style="border-bottom-color: black" v-model="selectedPeriod" @change="Period">
        <option value="current_month">Текущий месяц</option>
        <option value="current_week">Текущая неделя</option>
        <option value="today">Текущая сутка</option>
        <option value="period">Период</option>
      </select>

    </div>
  <div class="container">
    <div class="appointment-details">
      <h2>Средний чек</h2>
      <div id="chart"></div>
      <div id="labels" class="labels-container"></div>
    </div>
    <div class="appointment-details">
      <h2>Количество чеков</h2>
      <div id="count_check"></div>
      <div id="label_check" class="labels-container"></div>
    </div>
    <div class="charts">
      <div class="chart-wrapper">
        <h2 class="chart-heading">Касса</h2>
        <div class="programming-stats">
          <div class="chart-container">
            <canvas class="my-chart"></canvas>
          </div>
          <div class="details">
            <ul></ul>
          </div>
        </div>
      </div>
      <div class="chart-wrapper">
        <h2 class="chart-heading">Запись пациентов</h2>
        <div class="programming-stats">
          <div class="chart-container">
            <canvas class="services"></canvas>
          </div>
          <div class="service_details">
            <p>Количество всех записей: {{this.record.zapisi}}</p>
            <ul>
            </ul>
          </div>
        </div>
      </div>
    </div>
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
      selectedPeriod: "period",
      record: {
        allRecords : {},
        zapisi: null,
        count_for_tomorrow: null,
        count_for_today: null,
        came: null,
        accepted: null,
        rejected: null,
        awaited: null,
      },
      payments :{
        allPayments: {},
        total_by_check : null,
        total_paid: null,
        total_sum: null,
        count: null
      },
      doctors:{
        Doctors : [],
        count: null,
        doctorTotal: {},
        free_Time:{}
      }
    };
  },
  async mounted() {
    const promisesPayment = [];
    function filterPaymentsByDateRange(ResultQuery, startDate, endDate) {
      return ResultQuery.filter(payment => {
        const [day, month, year] = payment.date.split('.');
        const paymentDate = new Date(year, month - 1, day);
        return paymentDate >= startDate && paymentDate <= endDate;
      });
    }
    for (let i = 1; i <= 7; i++) {
      promisesPayment.push(axios.get(`https://api-developer.macdent.kz/payment/find/?page=${i}&access_token=${API_KEY}`));
    }
    const responsePayment = await Promise.all(promisesPayment);
    const ResPayments = responsePayment.flatMap(response=>{
      return response.data.pays
    })
    // console.log("allPayments-2022-2024: ",ResPayments)

    const today = new Date()
    const startDate = new Date(2023, 0, 1);
    const endDate = new Date(2024, 11, 31); // Например, на неделю вперед
    const StartMonth = new Date(today.getFullYear(), today.getMonth()-1, today.getDate())
    const endMonth = new Date(today.getFullYear(), today.getMonth(), today.getDate())
    const startWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate()-7);
    const endWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate());
    const startDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
    const endDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
    const startTomorrow  = new Date(today.getFullYear(), today.getMonth(), today.getDate()+1);
    const endTomorrow = new Date(today.getFullYear(), today.getMonth(), today.getDate()+1);
    this.payments.allPayments = filterPaymentsByDateRange(ResPayments, startDate, endDate)
    const paymentsByMonth = filterPaymentsByDateRange(this.payments.allPayments, StartMonth, endMonth)
    const paymentsByWeek = filterPaymentsByDateRange(this.payments.allPayments, startWeek, endWeek)
    const paymentsToday = filterPaymentsByDateRange(this.payments.allPayments, startDay, endDay)
    // console.log("AllPayments: ", this.payments.allPayments)
    // console.log("getByMonth: ", paymentsByMonth)
    // console.log("getByDay: " , paymentsToday)
    // console.log("getByWeek: ", paymentsByWeek)
    try {
      const promises = [];
      for (let i = 1; i <= 72; i++) {
        promises.push(axios.get(`https://api-developer.macdent.kz/zapis/find/?page=${i}&access_token=${API_KEY}`));
      }
      const responses = await Promise.all(promises);
      const Records = responses.flatMap(response => {
        return response.data.zapisi;
      });
      this.record.allRecords = filterPaymentsByDateRange(Records, startDate, endDate)
      if (this.selectedPeriod === "period"){
        this.processRecords(this.record.allRecords, startDate, endDate)
        this.main(this.payments.allPayments, startDate, endDate)
        this.fetchData(startDate, endDate)
      }

      this.record.count_for_tomorrow= filterPaymentsByDateRange(this.record.allRecords, startTomorrow, endTomorrow)
      this.record.count_for_today = filterPaymentsByDateRange(this.record.allRecords, startDay, endDay)
      // console.log("zapis for today: ",this.record.count_for_today)
      // console.log("zapis for tomorrow: ",this.record.count_for_tomorrow)
    } catch (error) {
      // console.error("An error occurred while fetching data:", error);
    }




  },
  methods:{
    filterPaymentsByDateRange(ResultQuery, startDate, endDate) {
  return ResultQuery.filter(payment => {
    const [day, month, year] = payment.date.split('.');
    const paymentDate = new Date(year, month - 1, day);
    return paymentDate >= startDate && paymentDate <= endDate;
  });
},
    processRecords(allRecord, startDate, endDate) {
      try {
        // Уничтожаем предыдущий график, если он существует
        if (this.chart) {
          this.chart.destroy();
        }

        const allRecords = this.filterPaymentsByDateRange(allRecord, startDate, endDate);
        this.record.came = allRecords.filter(record => record.status === 3 || record.status === 4 || record.status === 5 || record.status === 6);
        this.record.rejected = allRecords.filter(record => record.status === 2);
        this.record.accepted = allRecords.filter(record => record.status === 1);
        this.record.awaiting = allRecords.filter(record => record.status === 0);
        let came = this.record.came.length;
        let reject = this.record.rejected.length;
        let accepted = this.record.accepted.length;
        let awaiting = this.record.awaiting.length;

        const ServiceData = {
          labels: ["количество пришедших", "количество отклонивших", "количество подтвердивших", "количество в ожиданиях"],
          data: [came, reject, accepted, awaiting],
        };

        const services = document.querySelector(".services");
        const serv_ul = document.querySelector(".programming-stats .service_details ul");

        // Удаляем старые метки перед добавлением новых
        serv_ul.innerHTML = "";
        this.record.zapisi = allRecords.length;
        // Создаем новый график
        this.chart = new Chart(services, {
          type: "doughnut",
          data: {
            labels: ServiceData.labels,
            datasets: [
              {
                label: "сервисы",
                data: ServiceData.data,
                backgroundColor: [
                  "#36A2EB",
                  "#FF6384",
                  "#FFCE56",
                ],
              },
            ],
          },
          options: {
            borderWidth: 0,
            hoverBorderWidth: 0,
            plugins: {
              legend: {
                display: false,
              },
            },
          },
        });

        const populateServUl = () => {
          ServiceData.labels.forEach((l, i) => {
            let li = document.createElement("li");
            li.innerHTML = `${l}: <span class='percentage'>${ServiceData.data[i]}</span>`;
            serv_ul.appendChild(li);
          });
        };

        populateServUl();

        return ServiceData;
      } catch (error) {
        // console.error("An error occurred while processing data:", error);
        return null;
      }
    },
    async getDoctors(){
      this.doctors.Doctors = []
      this.doctors.Doctors.push(axios.get(`https://api-developer.macdent.kz/doctor/find/?access_token=${API_KEY}`));
      const DoctorInfo = await Promise.all(this.doctors.Doctors)
     let allDoctor = DoctorInfo.flatMap(response=>response.data.doctors)
      return allDoctor
    },
    async processPayment(AllPayment, startDate, endDate) {
      if (this.canvas) {
        this.canvas.destroy();
      }
      const myChart = document.querySelector(".my-chart");
      const existingChart = Chart.getChart(myChart);
      if (existingChart) {
        existingChart.destroy();
      }

      this.payments.total_by_check = 0
      this.payments.total_paid = 0
      this.payments.total_sum = 0
      let allDoctor = {};
      allDoctor = await this.getDoctors();

      // console.log("allDoctor in Func", allDoctor)
      allDoctor.forEach(doctor => {
        this.doctors.doctorTotal[doctor.id] = 0;
      });
      const allPayments = this.filterPaymentsByDateRange(AllPayment, startDate, endDate)
      // console.log("allPayments in function ", allPayments)
      allDoctor.forEach(doctor => {
        this.doctors.doctorTotal[doctor.id] = 0;
      });

      this.payments.count = allPayments.length
      let promisesDetailPayment = [];
      for(const pays of allPayments){
        this.payments.total_sum += pays.summ
        promisesDetailPayment.push(axios.get(`https://api-developer.macdent.kz/payment/get_detailed?id=${pays.id}&access_token=${API_KEY}`));
      }
      try {
        // console.log(promisesDetailPayment)
        const detailInfo = await Promise.all(promisesDetailPayment);
        // console.log(detailInfo)
        const allDetail = detailInfo.flatMap(response => response.data.pay)
        // console.log(allDetail)
        for (const detail of allDetail) {
          this.payments.total_by_check += parseInt(detail.totalByCheck);
          this.payments.total_paid += parseInt(detail.totalPaid);
        }
        // console.log("total by check after change",this.payments.total_by_check)
        // console.log("totalPaid after change",this.payments.total_paid)
        allDetail.forEach(pay => {
          const doctorId = pay.doctor;
          const paymentSum = parseInt(pay.totalPaid);

          if (this.doctors.doctorTotal.hasOwnProperty(doctorId)) {
            this.doctors.doctorTotal[doctorId] += paymentSum;
          }
        });

        let chartData = {
          labels: ["total_by_check", "total_Paid", "totalSum"],
          data: [this.payments.total_by_check, this.payments.total_paid, this.payments.total_sum],
        };
        // console.log(chartData)
        const avg_check = this.payments.total_by_check/this.payments.count
        // const avg_paid=this.payments.total_paid/this.payments.count
        // const avg_sum = this.payments.total_sum/this.payments.count
        // console.log(avg_check)
        const data = [
          { label: 'Средний чек', value: avg_check },
          { label: 'План', value: 30000 },
        ];
        const check_data = [
          { label: 'Количество чеков', value:this.payments.count},
          { label: 'План', value: 30 },
        ]
        // Chart 1

        const chart = document.getElementById('chart');
        chart.innerHTML = "";
        const labelsContainer = document.getElementById('labels');
        labelsContainer.innerHTML = ""
        const colors = ['blue', 'red', 'yellow', 'green'];
        labelsContainer.innerHTML = "";
        const maxValue = Math.max(...data.map(item => item.value));

        // Максимальная высота столбца (в пикселях)
        const maxHeight = 200; // Вы можете изменить это значение на ваше усмотрение

        data.forEach((item, index) => {
          const bar = document.createElement('div');
          bar.className = 'bar';
          bar.style.backgroundColor = colors[index % colors.length]; // Выбор цвета из массива в соответствии с индексом элемента

          const normalizedHeight = (item.value / maxValue) * maxHeight; // Нормализуем высоту столбца
          bar.style.height = `${normalizedHeight}px`; // Устанавливаем нормализованную высоту

          const span = document.createElement('span');
          span.textContent = item.value;

          bar.appendChild(span);
          chart.appendChild(bar);

          // Добавляем метки внизу диаграммы
          const label = document.createElement('div');
          label.textContent = item.label;
          labelsContainer.appendChild(label);
        });

        // Chart 2
        const chart_check = document.getElementById('count_check');
        chart_check.innerHTML= ""
        const labels_check = document.getElementById('label_check');
        labels_check.innerHTML = ""
        const checkValue = Math.max(...check_data.map(item => item.value));

        check_data.forEach((item, index) => {
          const bar = document.createElement('div');
          bar.className = 'bar';
          bar.style.backgroundColor = colors[index % colors.length];

          const normalizedHeight = (item.value / checkValue) * maxHeight;
          bar.style.height = `${normalizedHeight}px`;

          const span = document.createElement('span');
          span.textContent = item.value;

          bar.appendChild(span);
          chart_check.appendChild(bar);

          // Добавляем метки внизу диаграммы
          const label = document.createElement('div');
          label.textContent = item.label;
          labels_check.appendChild(label);
        });

        // Chart 3
        const myChart = document.querySelector(".my-chart");
        const ul = document.querySelector(".programming-stats .details ul");

        ul.innerHTML = "";
        new Chart(myChart, {
          type: "doughnut",
          data: {
            labels: chartData.labels,
            datasets: [
              {
                label: "Частота посещение",
                data: chartData.data,
                backgroundColor: [
                  "#36A2EB",
                  "#FF6384",
                  "#FFCE56",
                  "#4BC0C0",
                  "#9966FF"
                ],
              },
            ],
          },
          options: {
            borderWidth: 0,
            hoverBorderWidth: 0,
            plugins: {
              legend: {
                display: false,
              },
            },
          },
        });

        const populateUl = () => {
          chartData.labels.forEach((l, i) => {
            let li = document.createElement("li");
            li.innerHTML = `${l}: <span class='percentage'>${chartData.data[i]}</span>`;
            ul.appendChild(li);
          });
        };

        populateUl();

        // Chart 4
        return chartData
      } catch (error) {
        // console.error("An error occurred while fetching detailed payment information:", error);
      }
    },
    async main(allPayments, startDate, endDate) {
      try {
        const result = await this.processPayment(allPayments, startDate, endDate);
        return result

      } catch (error) {
        // console.error("An error occurred:", error);
      }
    },
    filterDoctorByDateRange(ResultQuery, startDate, endDate) {
  return ResultQuery.filter(free_time => {
    const dateParts = free_time.from.split(' ')[0].split('.'); // Разбиваем строку по пробелу, чтобы получить только дату
    const [day, month, year] = dateParts;

    if (!this.isValidDate(year, month, day)) {
      // console.error(`Invalid date: ${free_time.from}`);
      return false; // Пропускаем недействительные даты
    }
    const paymentDate = new Date(year, month - 1, day);
    return paymentDate >= startDate && paymentDate <= endDate;
  });
},

// Функция для проверки корректности даты
  isValidDate(year, month, day) {
    const date = new Date(year, month - 1, day);
    return !isNaN(date.getTime());
  },

async fetchData(startDate, endDate) {
  try {
    const doctor = {};
    let allDoctor = {};
    allDoctor = await this.getDoctors();
    // console.log("allDoctor",allDoctor)
    for (const doct of allDoctor) {
      // console.log(doct.id)
      const response = await axios.get(`https://api-developer.macdent.kz/doctor/get_free_time/?doctor=${doct.id}&access_token=${API_KEY}`);
      const Response = response.data.schedules;

      // Фильтруем расписание врачей по диапазону дат
      const FilterResponse = this.filterDoctorByDateRange(Response, startDate, endDate);

      let hours = 0; // Перемещаем объявление переменной сюда
      const doctorSchedule = FilterResponse.map(schedule => {
        const datePartsFrom = schedule.from.split(' ')[0].split('.').reverse(); // Разбиваем строку по пробелу и точке и меняем порядок на "год.месяц.день"
        const timePartsFrom = schedule.from.split(' ')[1]; // Получаем часть времени
        const dateFrom = new Date(datePartsFrom.join('-') + 'T' + timePartsFrom); // Объединяем дату и время в формате "год-месяц-деньTчасы:минуты:секунды"

        const datePartsTo = schedule.to.split(' ')[0].split('.').reverse();
        const timePartsTo = schedule.to.split(' ')[1];
        const dateTo = new Date(datePartsTo.join('-') + 'T' + timePartsTo);

        if (isNaN(dateFrom.getTime()) || isNaN(dateTo.getTime())) {
          // console.error(`Invalid date: ${schedule.from} or ${schedule.to}`);
          return null;
        }

        const differenceInMillis = dateTo - dateFrom;

        if (isNaN(differenceInMillis)) {
          // console.error(`Invalid time difference: ${schedule.from} to ${schedule.to}`);
          return null;
        }

        const hour = Math.floor(differenceInMillis / (1000 * 60 * 60));
        hours += hour;
        // console.log(hour)
        return { from: schedule.from, to: schedule.to };
      }).filter(schedule => schedule !== null);

      // Сохраняем расписание и количество часов для каждого врача
      doctor[doct.id] = { doctorSchedule, hours };
      // console.log(`doctors id ${doct.id}: `, hours);
    }
    // console.log("doctor: ",doctor);
    return doctor;
  } catch (error) {
    // console.error("Error fetching data for doctors:", error);
  }
},
    Period() {
      const today = new Date();
      const startDate = new Date(2023, 0, 1);
      const endDate = new Date(2024, 11, 31);
      const StartMonth = new Date(today.getFullYear(), today.getMonth() - 1, today.getDate());
      const endMonth = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      const startWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate() - 7);
      const endWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      const startDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      const endDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      // const startTomorrow = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 1);
      // const endTomorrow = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 1);

      if (this.selectedPeriod == "period") {
         this.processRecords(this.record.allRecords, startDate, endDate);
        this.main(this.payments.allPayments,startDate, endDate)
        // console.log("period", this.record)
      } else if (this.selectedPeriod == "current_month") {
         this.processRecords(this.record.allRecords, StartMonth, endMonth);
         this.main(this.payments.allPayments, StartMonth, endMonth)
        // console.log("month", this.record)
      } else if (this.selectedPeriod == "current_week") {
         this.processRecords(this.record.allRecords, startWeek, endWeek);
        this.main(this.payments.allPayments, startWeek, endWeek)
        // console.log("week", this.record)
      } else if (this.selectedPeriod == "today") {
         this.processRecords(this.record.allRecords, startDay, endDay);
        this.main(this.payments.allPayments, startDay, endDay)
        // console.log("day", this.record)
      }
    },
    watch: {
      selectedPeriod() {
        this.Period(); // Вызываем метод Period при изменении значения selectedPeriod
      },
    },
  }
}
</script>
<style>
.container {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  min-height: 100vh;
  margin-left: 5%;
}
.text{
  margin-left: 25%;
}
.text>p{
  margin-left: 15%;
}
.appointment-details {
  width: 30%;
  padding-right: 20px;
}

.charts {
  width: 65%;
}

.chart-wrapper {
  margin-top: 20px;
}

.chart-heading {
  font-family: "Rubik", sans-serif;
  color: #023047;
  text-transform: uppercase;
  font-size: 24px;
  text-align: center;
}

.chart-container {
  width: 200px;
}

.programming-stats {
  font-family: "Rubik", sans-serif;
  display: flex;
  align-items: center;
  gap: 24px;
  color: #023047;
}

.programming-stats .details ul {
  list-style: none;
  padding: 0;
}

.programming-stats .details ul li {
  font-size: 16px;
  margin: 12px 0;
  text-transform: uppercase;
}

.programming-stats .details .percentage {
  font-weight: 700;
  color: #e63946;
}

.my-chart,
.services {
  width: 200px;
  height: 200px;
}

.charts {
  width: 65%;
  margin-left: 5%;
}

#chart {
  width: 100%;
  height: 400px;
  border: 1px solid #ccc;
  display: flex;
  justify-content: space-around;
  align-items: flex-end;
  padding-bottom: 10px; /* Отступ снизу для красоты */
}

#chart .bar {
  width: 30px;
  border-right: 1px solid #ccc;
  box-sizing: border-box;
  animation: growBar 1s ease; /* Анимация увеличения высоты столбца */
}

#chart .bar span {
  display: block;
  text-align: center;
}

#count_check {
  width: 100%;
  height: 400px;
  border: 1px solid #ccc;
  display: flex;
  justify-content: space-around;
  align-items: flex-end;
  padding-bottom: 10px; /* Отступ снизу для красоты */
}

#count_check .bar {
  width: 30px;
  border-right: 1px solid #ccc;
  box-sizing: border-box;
  animation: growBar 1s ease; /* Анимация увеличения высоты столбца */
}

#count_check .bar span {
  display: block;
  text-align: center;
}


@keyframes growBar {
  from {
    height: 0;
  }
  to {
    height: 100%;
  }
}
/* Стили для контейнера с метками */
.labels-container {
  display: flex;
  justify-content: space-between;
  margin-top: 10px; /* Расстояние между диаграммой и метками */
}


</style>



<!--
    // function groupPaymentsByDate(payments, groupingFunction) {
    //   const groupedPayments = {};
    //   payments.forEach(payment => {
    //     const group = groupingFunction(payment.date);
    //     if (!groupedPayments[group]) {
    //       groupedPayments[group] = [];
    //     }
    //     groupedPayments[group].push(payment);
    //   });
    //   return groupedPayments;
    // }
//     const paymentsByMonth = groupPaymentsByDate(allPayments, payment => {
//       const [day, month, year] = payment.split('.'); // Разбиваем строку на день, месяц и год
//       const date = new Date(year, month - 1, day); // Месяцы в JavaScript начинаются с 0
//       return date.getMonth();
//     });
//
//
// // Группировка платежей по неделям
//     const paymentsByWeek = groupPaymentsByDate(allPayments, payment => {
//       const [day, month, year] = payment.split('.'); // Разбиваем строку на день, месяц и год
//       const date = new Date(year, month - 1, day); // Месяцы в JavaScript начинаются с 0
//       const startOfWeek = new Date(date);
//       startOfWeek.setDate(date.getDate() - date.getDay());
//       return startOfWeek.toISOString();
//     });
//
//     const today = new Date();
//     const todayString = today.toLocaleDateString('en-US', { year: 'numeric', month: '2-digit', day: '2-digit' });
//
//     const paymentsToday = allPayments.filter(payment => {
//       const [day, month, year] = payment.date.split('.');
//       const paymentDate = new Date(year, month - 1, day);
//       const paymentDateString = paymentDate.toLocaleDateString('en-US', { year: 'numeric', month: '2-digit', day: '2-digit' });
//       return paymentDateString === todayString;
//     });

        // let found = false;
        // AllDoctor.forEach(doctor => {
        //   if (detail.doctor === doctor.id) {
        //     console.log("success");
        //     found = true;
        //   }
        // })
        // if (!found) {
        //   console.log("!doctor pay: ",detail);
        // }

-->