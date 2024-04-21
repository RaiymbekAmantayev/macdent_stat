<template>
<!-- <h1>STATIC</h1>-->
<!--  <h1>Аналитика данных</h1>-->
  <section class="main">
    <table>
      <tr>
        <th>Названия</th>
        <th>День</th>
        <th>Неделя</th>
        <th>Месяц</th>
      </tr>
      <tr>
        <td>Выручка</td>
        <td>{{payments.total_paid_day}}</td>
        <td>{{payments.total_paid_week}}</td>
        <td>{{payments.total_paid_month}}</td>
      </tr>
      <tr>
        <td>количество записей на сегодня</td>
        <td>{{record.count_for_today}}</td>
        <td>{{record.count_week}}</td>
        <td>{{record.count_month}}</td>
      </tr>
      <tr>
        <td>Количество пришедших</td>
        <td>{{record.came_day}}</td>
        <td>{{record.came_week}}</td>
        <td>{{record.came_month}}</td>
      </tr>
      <tr>
        <td>Количество отклонивших</td>
        <td>{{record.rejected_day}}</td>
        <td>{{record.rejected_week}}</td>
        <td>{{record.rejected_month}}</td>
      </tr>
      <tr>
        <td>количество записей на завтра</td>
        <td>{{record.count_for_tomorrow}}</td>
      </tr>
      <tr>
        <td>средний чек</td>
        <td>{{payments.avg_paid_day}}</td>
        <td>{{payments.avg_paid_week}}</td>
        <td>{{ payments.avg_paid_month }}</td>
      </tr>
      <tr>
        <td>Kaspi pay</td>
        <td>{{payments.kaspiPay_day}}</td>
        <td>{{payments.kaspiPay_week}}</td>
        <td>{{payments.kaspiPay_month}}</td>
      </tr>
      <tr>
        <td>Наличные</td>
        <td>{{payments.cash_day}}</td>
        <td>{{payments.cash_week}}</td>
        <td>{{payments.cash_month}}</td>
      </tr>
      <tr>
        <td>Количество обращении общее</td>
        <td>{{this.whereKnow.allRef_day}}</td>
        <td>{{this.whereKnow.allRef_week}}</td>
        <td>{{this.whereKnow.allRef_month}}</td>
      </tr>
      <tr>
        <td>2гис</td>
        <td>{{this.patients.whereKnow.two_gis.day}}</td>
        <td>{{this.patients.whereKnow.two_gis.week}}</td>
        <td>{{this.patients.whereKnow.two_gis.month}}</td>
      </tr>
      <tr>
        <td>интернет</td>
        <td>{{this.patients.whereKnow.internet.day}}</td>
        <td>{{this.patients.whereKnow.internet.week}}</td>
        <td>{{this.patients.whereKnow.internet.month}}</td>
      </tr>
      <tr>
        <td>Рекомендации</td>
        <td>{{this.patients.whereKnow.recomend.day}}</td>
        <td>{{this.patients.whereKnow.recomend.week}}</td>
        <td>{{this.patients.whereKnow.recomend.month}}</td>
      </tr>
      <tr>
        <td>Старая база</td>
        <td>{{this.patients.whereKnow.old_db.day}}</td>
        <td>{{this.patients.whereKnow.old_db.week}}</td>
        <td>{{this.patients.whereKnow.old_db.month}}</td>
      </tr>
    </table>
    <div class="select__wrapper">
      <div class="select__wrapper-text">
        За
      </div>
      <select v-model="selectedValue" @change="updateChart">
        <option value="day">Текущий день</option>
        <option value="week">Текущая неделя</option>
        <option value="month">Текущий месяц</option>
        <option value="custom">За период</option>
      </select>
    </div>
    <div class="about">
      <div class="about__content">
        <div class="about__content-text">
          Выручка
        </div>
        <div class="about__content-price">
          {{currentValue.total_paid}}
        </div>
      </div>
      <hr>
      <div class="about__content">
        <div class="about__content-text">
          Средний чек
        </div>
        <div class="about__content-price">
          {{currentValue.avg_paid}}
        </div>
      </div>
      <hr>
      <div class="about__content">
        <div class="about__content-text">
          Количество продаж
        </div>
        <div class="about__content-price">
          {{currentValue.count}}
        </div>
      </div>
      <hr>
      <div class="about__content">
        <div class="about__content-text">
          Себестоимость
        </div>
        <div class="about__content-price">
          0
        </div>
      </div>
      <hr>
      <div class="about__content">
        <div class="about__content-text">
          Валовая прибыль
        </div>
        <div class="about__content-price">
          0
        </div>
      </div>
      <hr>
    </div>


    <input type="date" id="startDate" class="custom-input" style="display:none;">
    <input type="date" id="endDate" class="custom-input" style="display:none;">

    <canvas id="myChart"></canvas>
  </section>
</template>

<script>
import Chart from 'chart.js/auto';
import axios from "axios";
import {API_KEY} from "@/pages/API_KEY";

export default {
  data() {
    return {
      record: {
        allRecords : {},
        zapisi: null,
        count_for_tomorrow: null,
        count_for_today: null,
        count_week: null,
        count_month: null,
        came_day: null,
        came_week:null,
        came_month:null,
        accepted: null,
        rejected_day: null,
        rejected_week:null,
        rejected_month:null,
        awaited: null,
      },
      payments :{
        allDetail: {},
        allPayments: {},
        total_paid_day: null,
        total_paid_week: null,
        total_paid_month: null,
        avg_paid_day:null,
        avg_paid_week:null,
        avg_paid_month:null,
        kaspiPay_month: null,
        kaspiPay_week: null,
        kaspiPay_day: null,
        cash_month: null,
        cash_week:null,
        cash_day:null,
        count_day: null,
        count_week:null,
        count_month: null
      },
      patients:{
        whereKnow: {
          two_gis: {
            month: null,
            week: null,
            day: null
          },
          internet: {
            month: null,
            week: null,
            day: null
          },
          recomend: {
            month: null,
            week: null,
            day: null
          },
          old_db: {
            month: null,
            week: null,
            day: null
          },
        },
      },
      whereKnow:{
        allRef_month:null,
        allRef_week: null,
        allRef_day: null
      },
      selectedValue: 'day',
      myChart: null,
      currentValue:{
        total_paid: null,
        avg_paid: null,
        count: null
      },
      labels: [],
      data:[]
    };
  },
  async mounted() {
    this.initializeChart();
    this.toggleDateInputs();
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
    // let whereKnows = axios.get(`https://api-developer.macdent.kz/where_know/find/?access_token=${API_KEY}`)
    // this.whereKnow = whereKnows.data.items
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
    // console.log("AllPayments: ", this.payments.allPayments)

    const monthResult = await this.getTotalPaid.call(this, this.payments.allPayments, StartMonth, endMonth);
    this.payments.total_paid_month = monthResult.total_paid;
    this.payments.avg_paid_month = monthResult.total_avg;
    this.payments.count_month = monthResult.count;
    this.payments.kaspiPay_month = monthResult.kaspiPay
    this.payments.cash_month = monthResult.Cash
    this.patients.whereKnow.two_gis.month= monthResult.two_gis
    this.patients.whereKnow.internet.month = monthResult.internet
    this.patients.whereKnow.recomend.month = monthResult.rec
    this.patients.whereKnow.old_db.month = monthResult.old_db
    this.whereKnow.allRef_month = monthResult.allRef
    const weekResult = await this.getTotalPaid.call(this, this.payments.allPayments, startWeek, endWeek);
    this.payments.total_paid_week = weekResult.total_paid;
    this.payments.avg_paid_week = weekResult.total_avg;
    this.payments.count_week = weekResult.count;
    this.payments.kaspiPay_week = weekResult.kaspiPay
    this.payments.cash_week = weekResult.Cash
    this.patients.whereKnow.two_gis.week= weekResult.two_gis
    this.patients.whereKnow.internet.week = weekResult.internet
    this.patients.whereKnow.recomend.week = weekResult.rec
    this.patients.whereKnow.old_db.week = weekResult.old_db
    this.whereKnow.allRef_week = weekResult.allRef
    const dayResult = await this.getTotalPaid.call(this, this.payments.allPayments, startDay, endDay);
    this.payments.total_paid_day = dayResult.total_paid;
    this.payments.avg_paid_day = dayResult.total_avg;
    this.payments.count_day = dayResult.count;
    this.payments.kaspiPay_day = dayResult.kaspiPay
    this.payments.cash_day = dayResult.Cash
    this.patients.whereKnow.two_gis.day= dayResult.two_gis
    this.patients.whereKnow.internet.day = dayResult.internet
    this.patients.whereKnow.recomend.day = dayResult.rec
    this.patients.whereKnow.old_db.day = dayResult.old_db
    this.whereKnow.allRef_day = dayResult.allRef
    if(this.selectedValue == "day") {
      this.currentValue.total_paid = dayResult.total_paid
      this.currentValue.avg_paid = dayResult.total_avg
      this.currentValue.count = dayResult.count
      const paymentsByDay_InDay = this.groupPaymentsByDate(this.payments.allDetail, payment => {
        const [day, month, year] = payment.split('.');
        const formattedDate = new Date(`${year}-${month}-${day}`);
        const yearMonthDay = formattedDate.toISOString().split('T')[0]; // Получаем год, месяц и день в формате 'гггг-мм-дд'
        return yearMonthDay;
      });
      const key_day = Object.keys(paymentsByDay_InDay)
      const value_day = Object.values(paymentsByDay_InDay)
      let totalPaidByArray_InDay = [];
      value_day.forEach(paymentsArray => {
        let totalPaid = 0;
        paymentsArray.forEach(payment => {
          totalPaid += parseInt(payment.totalPaid);
        });
        totalPaidByArray_InDay.push(totalPaid);
      });
      // console.log("payments by day: ", paymentsByDay_InDay)
      this.data = totalPaidByArray_InDay;
      this.label = key_day
    }

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
      let tomorrow = filterPaymentsByDateRange(this.record.allRecords, startTomorrow, endTomorrow)
      this.record.count_for_tomorrow = tomorrow.length

      let today = filterPaymentsByDateRange(this.record.allRecords, startDay, endDay)
      let record_today = today.filter(record => record.status === 3 || record.status === 4 || record.status === 5 || record.status === 6);
      // console.log("today's rec", record_today)
      this.record.came_day = record_today.length
      let record_rej_day = today.filter(record => record.status === 2);
      this.record.count_for_today = today.length
      this.record.rejected_day = record_rej_day.length

      let week = filterPaymentsByDateRange(this.record.allRecords, startWeek, endWeek)
      let record_week = week.filter(record => record.status === 3 || record.status === 4 || record.status === 5 || record.status === 6);
      // console.log("week's rec: ", record_week)
      this.record.came_week = record_week.length
      let record_rej_week = week.filter(record => record.status === 2);
      this.record.count_week = week.length
      this.record.rejected_week = record_rej_week.length

      let month = filterPaymentsByDateRange(this.record.allRecords, StartMonth, endMonth)
      let record_month = month.filter(record => record.status === 3 || record.status === 4 || record.status === 5 || record.status === 6);
      this.record.came_month = record_month.length
      let record_rej_month = month.filter(record => record.status === 2);
      this.record.count_month = month.length
      this.record.rejected_month = record_rej_month.length

      // console.log("zapis for today: ",this.record.count_for_today)
      // console.log("zapis for tomorrow: ",this.record.count_for_tomorrow)
    } catch (error) {
      // console.error("An error occurred while fetching data:", error);
    }
  },
  methods: {
    filterPaymentsByDateRange(ResultQuery, startDate, endDate) {
      return ResultQuery.filter(payment => {
      const [day, month, year] = payment.date.split('.');
      const paymentDate = new Date(year, month - 1, day);
      return paymentDate >= startDate && paymentDate <= endDate;
    });
  },
    async getTotalPaid(allPayments, startDate, endDate){
      let promisesDetailPayment = [];
      let patiensList = []
      let kaspiPay = 0
      let Cash = 0
      let allPayment = this.filterPaymentsByDateRange(allPayments, startDate, endDate)
      for(const pays of allPayment){
        if (pays.typeOplata && pays.typeOplata.length > 0) {
          let typePay = pays.typeOplata[0];
          if (typePay.system == "Kaspi Pay") {
            kaspiPay += 1;
          }else if(typePay.system == "Наличные"){
            Cash+=1
          }else {
            // console.error("typeOplata is undefined or empty for payment:", pays);
          }
        }
        patiensList.push(axios.get(`https://api-developer.macdent.kz/patient/get?id=${pays.patient}&access_token=${API_KEY}`))
        promisesDetailPayment.push(axios.get(`https://api-developer.macdent.kz/payment/get_detailed?id=${pays.id}&access_token=${API_KEY}`));
      }
      // about detailPatients
      const detailPatient = await Promise.all(patiensList)
      const allPatient = detailPatient.flatMap(response=>response.data.patient)
      let two_gis = 0
      let internet = 0
      let rec = 0
      let old_db = 0
      let allRef = 0
      for (let patient of allPatient) {
        if (patient.whereKnow) {
            allRef+=1
          // Проверяем значение
          if (patient.whereKnow === "2 гис") {
            two_gis += 1;
          } else if (patient.whereKnow === "Интернет") {
            internet += 1;
          } else if (patient.whereKnow === "Рекомендация") {
            rec += 1;
          } else if (patient.whereKnow === "Старая база") {
            old_db += 1;
          }
        }
      }
      // about detailPayment
      const detailInfo = await Promise.all(promisesDetailPayment);
      const allDetail = detailInfo.flatMap(response => response.data.pay)
      this.payments.allDetail = allDetail
      // console.log(allDetail)
      let total_paid = 0
      let count = 0
      let total_avg = 0
      count = allDetail.length
      for (const detail of allDetail) {
        // this.payments.total_by_check += parseInt(detail.totalByCheck);
        total_paid += parseInt(detail.totalPaid);
      }
      // console.log("result:", total_paid)
      // this.payments.count = allPayment.length
      // this.payments.avg_paid = total_paid/this.payments.count
      total_avg = total_paid/count
      return { total_paid, total_avg, count, kaspiPay, Cash, two_gis, internet, rec, old_db, allRef };
    },
    groupPaymentsByDate(payments, groupingFunction) {
      const groupedPayments = {};
      payments.forEach(payment => {
        const group = groupingFunction(payment.date);
        if (!groupedPayments[group]) {
          groupedPayments[group] = [];
        }
        groupedPayments[group].push(payment);
      });
      return groupedPayments;
    },
    async updateChart() {
      const today = new Date()
      const startDate = new Date(2023, 0, 1);
      const endDate = new Date(2024, 11, 31); // Например, на неделю вперед
      const StartMonth = new Date(today.getFullYear(), today.getMonth()-1, today.getDate())
      const endMonth = new Date(today.getFullYear(), today.getMonth(), today.getDate())
      const startWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate()-7);
      const endWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      const startDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      const endDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      var data;
      let label;
      switch (this.selectedValue) {
        case 'day':
          const dayResult = await this.getTotalPaid.call(this, this.payments.allPayments, startDay, endDay);
          this.currentValue.total_paid = dayResult.total_paid
          this.currentValue.avg_paid = dayResult.total_avg
          this.currentValue.count = dayResult.count

          const paymentsByDay_InDay = this.groupPaymentsByDate(this.payments.allDetail, payment => {
            const [day, month, year] = payment.split('.');
            const formattedDate = new Date(`${year}-${month}-${day}`);
            const yearMonthDay = formattedDate.toISOString().split('T')[0]; // Получаем год, месяц и день в формате 'гггг-мм-дд'
            return yearMonthDay;
          });
          const key_day = Object.keys(paymentsByDay_InDay)
          const value_day = Object.values(paymentsByDay_InDay)
          let totalPaidByArray_InDay= [];
          value_day.forEach(paymentsArray => {
            let totalPaid = 0;
            paymentsArray.forEach(payment => {
              totalPaid += parseInt(payment.totalPaid);
            });
            totalPaidByArray_InDay.push(totalPaid);
          });
          // console.log("payments by day: ", paymentsByDay)
          data = totalPaidByArray_InDay;
          label = key_day
          break;
        case 'week':
          const weekResult = await this.getTotalPaid.call(this, this.payments.allPayments, startWeek, endWeek);
            this.currentValue.total_paid = weekResult.total_paid
            this.currentValue.avg_paid = weekResult.total_avg
            this.currentValue.count = weekResult.count
            // console.log("week: ",this.currentValue.total_paid)

          const paymentsByDay = this.groupPaymentsByDate(this.payments.allDetail, payment => {
            const [day, month, year] = payment.split('.');
            const formattedDate = new Date(`${year}-${month}-${day}`);
            const yearMonthDay = formattedDate.toISOString().split('T')[0]; // Получаем год, месяц и день в формате 'гггг-мм-дд'
            return yearMonthDay;
          });
          const dates_day = Object.keys(paymentsByDay)
          const values_day = Object.values(paymentsByDay)
          let totalPaidByArray_day = [];
          values_day.forEach(paymentsArray => {
            let totalPaid = 0;
            paymentsArray.forEach(payment => {
              totalPaid += parseInt(payment.totalPaid);
            });
            totalPaidByArray_day.push(totalPaid);
          });
            // console.log("payments by day: ", paymentsByDay)
          data = totalPaidByArray_day;
          label = dates_day
          break;
        case 'month':
          const monthResult = await this.getTotalPaid.call(this, this.payments.allPayments, StartMonth, endMonth);
            this.currentValue.total_paid = monthResult.total_paid
            this.currentValue.avg_paid = monthResult.total_avg
            this.currentValue.count = monthResult.count

          const paymentsByDayInMonth = this.groupPaymentsByDate(this.payments.allDetail, payment => {
            const [day, month, year] = payment.split('.');
            const formattedDate = new Date(`${year}-${month}-${day}`);
            const yearMonthDay = formattedDate.toISOString().split('T')[0]; // Получаем год, месяц и день в формате 'гггг-мм-дд'
            return yearMonthDay;
          });
          const dates_week = Object.keys(paymentsByDayInMonth)
          const values_week = Object.values(paymentsByDayInMonth)
          const totalPaidByArray = [];

          values_week.forEach(paymentsArray => {
            let totalPaid = 0;
            paymentsArray.forEach(payment => {
              totalPaid += parseInt(payment.totalPaid);
            });
            totalPaidByArray.push(totalPaid);
          });
          this.data = totalPaidByArray
          data = totalPaidByArray;
          label = dates_week
          break;
        case 'year':
          data = [200, 190, 180, 195, 205];
          break;
        case 'custom':
            const customResult = await this.getTotalPaid.call(this, this.payments.allPayments, startDate, endDate);
            this.currentValue.total_paid = customResult.total_paid
            this.currentValue.avg_paid = customResult.total_avg
            this.currentValue.count = customResult.count
          data = [50, 60, 70, 80, 90];
          break;
      }
      this.myChart.data.datasets[0].data = data;
      this.myChart.data.labels = label;
      this.myChart.update();


    },
    initializeChart() {
      var ctx = document.getElementById('myChart');
      this.myChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: this.labels,//['05.04-06.04', '06.04-07.04', '07.04-8.04', '08.04-09.04', '09.04-10.04', '10.04-11.04', '11.04-12.04', '12.04-12.04'],
          datasets: [{
            label: "",
            data: this.data, //[12, 19, 3, 5, 2, 10, 15],
            backgroundColor: 'rgba(255, 99, 132, 0.2)',
            borderColor: 'rgba(255, 99, 132, 1)',
            borderWidth: 1
          }]
        },
        options: {
          scales: {
            yAxes: [{
              ticks: {
                beginAtZero: false,
                min: 20,
                max: 200,
                stepSize: 50
              }
            }]
          }
        }
      });
    },
    toggleDateInputs() {
      var customOption = this.selectedValue;
      var startDateInput = document.getElementById('startDate');
      var endDateInput = document.getElementById('endDate');

      if (customOption === 'custom') {
        startDateInput.style.display = 'inline';
        endDateInput.style.display = 'inline';
      } else {
        startDateInput.style.display = 'none';
        endDateInput.style.display = 'none';
      }
    }
  }
}
</script>

<style scoped>
body {
  background-color: #f2f2f2;
  font-family: 'Roboto', sans-serif;
}

table {
  border-collapse: collapse;
  width: 900px;
  margin: 0 auto;
  background-color: #fff;
  margin-top: 15px;
  margin-bottom: 25px;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

.about {
  margin: 0 auto;
  max-width: 900px;
  background-color: #ffffff;

}

.about__content {
  display: flex;
  justify-content: space-between;
}

.about__content-text {
  margin-right: 50px;
  padding-left: 50px;
  padding-top: 10px;
  padding-bottom: 10px;
}

.about__content-price {
  padding-right: 50px;
  padding-top: 10px;
  padding-bottom: 10px;
}

hr {
  width: 900px;
  border: 1px solid rgb(224, 224, 224);
}

canvas {
  background-color: #ffffff;
}

#myChart {
  max-width: 900px;
  width: 100%;
  height: auto;
  margin: 0 auto;
}

.chart-container {
  width: 900px;
  margin: 0 auto;
}

.select__wrapper {
  display: flex;
  max-width: 900px;
  margin: 0 auto;
  padding-top: 25px;
  padding-bottom: 25px;
  align-items: center;
}

.select__wrapper-text {
  font-size: 20px;
  margin-right: 15px;
}

select {
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  padding: 8px 12px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #fff;
  cursor: pointer;
}

select:focus {
  outline: none;
  border-color: #66afe9;
  box-shadow: 0 0 5px rgba(102, 175, 233, 0.5);
}

select option {
  padding: 8px 12px;
}

.custom-input {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 8px 12px;
  font-size: 14px;
  width: 200px;
}

@media screen and (max-width: 800px) {

  table {
    width: 100%;
  }

  th,
  td {
    width: 100%;
  }

  th {
    text-align: center;
  }

  tr:nth-child(even) {
    background-color: #f2f2f2;
  }

  .about__content {
    flex-direction: column;
    text-align: center;
  }

  .about__content-text,
  .about__content-price {
    margin: 10px 0;
    padding: 0;
  }

  hr {
    width: 344px;
  }

  .select__wrapper {
    flex-direction: column;
    align-items: flex-start;
  }

  .select__wrapper-text {
    margin-bottom: 5px;
  }

  select {
    width: 100%;
  }

  option {
    max-width: 10px;
  }

  .custom-input {
    width: 100px;
    margin-bottom: 20px;
  }

  #myChart {
    max-width: 100%;
  }

  .chart-container {
    padding: 0 10px;
  }
}
</style>
