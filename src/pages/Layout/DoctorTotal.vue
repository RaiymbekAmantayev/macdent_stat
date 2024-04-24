<template>
  <div class="content">
    <div class="md-layout">
      <div class="md-layout-item md-medium-size-100 md-xsmall-size-100 md-size-33">
        <chart-card
            :chart-data="emailsSubscriptionChart.data"
            :chart-options="emailsSubscriptionChart.options"
            :chart-responsive-options="emailsSubscriptionChart.responsiveOptions"
            :chart-type="'Bar'"
            data-background-color="red"
        >
          <template slot="content">
            <h4 class="title">Email Subscription</h4>
            <p class="category">Last Campaign Performance</p>
          </template>

          <template slot="footer">
            <div class="stats">
              <md-icon>access_time</md-icon>
              updated 10 days ago
            </div>
          </template>
        </chart-card>
      </div>

      <div
          class="md-layout-item md-medium-size-100 md-xsmall-size-100 md-size-50"
      >
        <md-card>
          <md-card-header data-background-color="orange">
            <h4 class="title">Employees Stats</h4>
            <p class="category">New employees on 15th September, 2016</p>
          </md-card-header>
          <md-card-content>
            <ordered-table table-header-color="orange"></ordered-table>
          </md-card-content>
        </md-card>
      </div>
    </div>
  </div>
</template>

<script>
import {ChartCard, OrderedTable,} from "@/components";

export default {
  props: {
    doctorTotal: Object,
    busy_time:Object,
    countCheck:Object
  },
  data() {
    return {
      emailsSubscriptionChart: {
        data: {
          doctorTotals: {},
          labels: [
            "Ja",
            "Fe",
            "Ma",
            "Ap",
            "Mai",
            "Ju",
            "Jul",
            "Au",
            "Se",
            "Oc",
            "No",
            "De",
          ],
          series: [
            [542, 443, 320, 780, 553, 453, 326, 434, 568, 610, 756, 895],
          ],
        },
        options: {
          axisX: {
            showGrid: false,
          },
          low: 0,
          high: 1000,
          chartPadding: {
            top: 0,
            right: 5,
            bottom: 0,
            left: 0,
          },
        },
        responsiveOptions: [
          [
            "screen and (max-width: 640px)",
            {
              seriesBarDistance: 5,
              axisX: {
                labelInterpolationFnc: function (value) {
                  return value[0];
                },
              },
            },
          ],
        ],
      },
    };
  },
  components: {
    ChartCard,
    OrderedTable,
  },
  async mounted() {
    // console.log("records: ", this.doctorTotal);
    // console.log("busy_time_from_DrTT: ", this.busy_time)
    // console.log("countCheck: ", this.countCheck)
  },
  watch: {
    '$props.doctorTotal': {
      handler(newDoctorTotal, oldDoctorTotal) {
        console.log('Данные о докторах обновлены:', newDoctorTotal);
      },
      deep: true
    },
    '$props.busy_time':{
      handler(newBusyTime, oldBusyTime) {
        // console.log('Данные о записей обновлены:', newBusyTime);
      },
      deep:true
    },
    '$props.countCheck':{
      handler(newCount, oldCount) {
        // console.log('Данные о количестве чеков обновлены:', newCount);
      },
      deep:true
    }
  }
};
</script>
