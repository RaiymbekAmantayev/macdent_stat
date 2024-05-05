<template>
  <div>
  <h1>Res</h1>
  <div>
  <button @click="DayRes">day</button>
  </div>
  <div>
  <button @click="WeekRes">week</button>
  </div>
  <div>
  <button @click="MonthRes">month</button>
  </div>
  <div>
  <h2>{{result}}</h2>
  </div>
  </div>
</template>

<script>
export default {
  props: {
    allRecordsByPeriod: [],
  },
  data() {
    return {
      chartInstances: {}, // Хранение экземпляров графиков
      times:["07:00", "08.00", "09:00", "10:00", "11:00", "12:00", "13:00", "14:00", "15:00","16:00","17:00","18:00", "19:00","20:00"], // Добавлена запятая
      result: []
    };

  },
  mounted() {
    this.WeekRes()
  },
  watch: {
    allRecordsByPeriod(newCountCheck) {
      console.log("zapisi: ",newCountCheck)
    },
    result(newResult) {
      // Обновляем значение свойства result при его изменении
      this.result = newResult;
      console.log("Updated result: ", this.result);
    }
  },
  methods: {
    filterPaymentsByDateRange(ResultQuery , startDate, endDate) {
      return ResultQuery.filter(payment => {
        const [day, month, year] = payment.date.split('.');
        const paymentDate = new Date(year, month - 1, day);
        return paymentDate >= startDate && paymentDate <= endDate;
      });
    },
    async getTotalRecod(allRecords, startDate, endDate){
      let allRecord = this.filterPaymentsByDateRange(allRecords, startDate, endDate)
      let recordsByHour = {};
      allRecord.forEach(record => {
        let startTime = record.start;
        let startHour = startTime.split(' ')[1].split(':')[0];
        startHour = parseInt(startHour);
        let endTime = record.end;
        let endHour = endTime.split(' ')[1].split(':')[0];

        // Преобразуем в числа
        startHour = parseInt(startHour);
        endHour = parseInt(endHour);

        // Уменьшаем endHour на 1
        if (startHour === endHour) {
          console.log("start: ",startHour)
          console.log("end: ", endHour)
          const hourString = startHour < 10 ? "0" + startHour + ":00" : startHour + ":00";
          if (this.times.includes(hourString)) {
            if (!recordsByHour[hourString]) {
              recordsByHour[hourString] = 1;
            } else {
              recordsByHour[hourString]++;
            }
          }
        } else {
          endHour -= 1;
          for (let hour = startHour; hour <= endHour; hour++) {
            const hourString = hour < 10 ? "0" + hour + ":00" : hour + ":00";
            if (this.times.includes(hourString)) {
              if (!recordsByHour[hourString]) {
                recordsByHour[hourString] = 1;
              } else {
                recordsByHour[hourString]++;
              }
            }
          }
        }




      });
      this.result = recordsByHour
      console.log("result in function ", this.result);
    },
    async DayRes(){
      const today = new Date();
      const startDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      const endDay = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      await this.getTotalRecod(this.allRecordsByPeriod, startDay, endDay);
    },
    async WeekRes(){
      const today = new Date()
      const startWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate() - 7);
      const endWeek = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      await this.getTotalRecod(this.allRecordsByPeriod, startWeek, endWeek);
    },
    async MonthRes(){
      const today = new Date()
      const StartMonth = new Date(today.getFullYear(), today.getMonth() - 1, today.getDate());
      const endMonth = new Date(today.getFullYear(), today.getMonth(), today.getDate());
      await this.getTotalRecod(this.allRecordsByPeriod, StartMonth, endMonth);
    }
  }
}
</script>
<style>

</style>