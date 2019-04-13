<template>
  <div class="chartElem">
    <date-picker v-model="dateRange" range lang="en" format="YYYY-MM-DD" confirm ></date-picker>
    <highcharts class="chart" :options="chartOptions"></highcharts>
  </div>
</template>

<script>
import DatePicker from 'vue2-datepicker'
import moment from 'moment'

export default {
  components: {
    DatePicker
  },
  props: {
    trelloCards: Object
  },
  data () {
    return {
      startDate: new Date(),
      endDate: new Date(),
      dateRange: {},
      placeholder: '',
      chartOptions: {
        colors: ['#2b908f', '#90ee7e', '#f45b5b', '#7798BF', '#aaeeee', '#ff0066',
          '#eeaaee', '#55BF3B', '#DF5353', '#7798BF', '#aaeeee'],
        chart: {
          type: 'area'
        },
        title: {
          text: this.title
        },
        // subtitle: {
        //   text: 'Source: Wikipedia.org'
        // },
        xAxis: {
          categories: [],
          tickmarkPlacement: 'on',
          title: {
            enabled: false
          }
        },
        yAxis: {
          title: {
            text: 'Task Count'
          }
          // labels: {
          //   formatter: function () {
          //     return this.value / 1000
          //   }
          // }
        },
        // tooltip: {
        //   split: true,
        //   valueSuffix: ' millions'
        // },
        plotOptions: {
          area: {
            stacking: 'normal',
            lineColor: '#666666',
            lineWidth: 1,
            marker: {
              lineWidth: 1,
              lineColor: '#666666'
            }
          }
        },
        series: []
      }
    }
  },
  watch: {
    trelloCards (newValue) {
      this.trelloCards = newValue
      // set X axis
      this.chartOptions.xAxis.categories = []
      const days = moment(this.endDate).date() - moment(this.startDate).date()
      console.log(days)
      for (let index = 0; index <= days; index++) {
        console.log(moment(this.startDate))
        var date = moment(this.startDate).add(index, 'days').format('YYYY-MM-DD')
        console.log(date)
        this.chartOptions.xAxis.categories.push(date)
      }

      // set Y axis
      this.chartOptions.series = []
      for (const [name, userData] of this.trelloCards.entries()) {
        var serie = {}
        serie.name = name
        serie.data = []
        this.chartOptions.xAxis.categories.forEach(date => {
          if (userData.has(date)) {
            serie.data.push(userData.get(date).length)
          } else {
            serie.data.push(0)
          }
        })
        this.chartOptions.series.push(serie)
      }
    },
    dateRange () {
      console.log(this.dateRange)
      this.startDate = moment(this.dateRange[0]).format('YYYY-MM-DD')
      this.endDate = moment(this.dateRange[1]).format('YYYY-MM-DD')
    },
    startDate () {
      // set X axis
      this.chartOptions.xAxis.categories = []
      const days = moment(this.endDate).date() - moment(this.startDate).date()
      for (let index = 0; index <= days; index++) {
        var date = moment(this.startDate).add(index, 'days').format('YYYY-MM-DD')
        this.chartOptions.xAxis.categories.push(date)
      }

      // set Y axis
      this.chartOptions.series = []
      for (const [name, userData] of this.trelloCards.entries()) {
        var serie = {}
        serie.name = name
        serie.data = []
        this.chartOptions.xAxis.categories.forEach(date => {
          if (userData.has(date)) {
            serie.data.push(userData.get(date).length)
          } else {
            serie.data.push(0)
          }
        })
        this.chartOptions.series.push(serie)
      }
    },
    endDate () {
      // set X axis
      this.chartOptions.xAxis.categories = []
      const days = (moment(this.endDate).diff(moment(this.startDate))) / (1000 * 60 * 60 * 24)
      for (let index = 0; index <= days; index++) {
        var date = moment(this.startDate).add(index, 'days').format('YYYY-MM-DD')
        this.chartOptions.xAxis.categories.push(date)
      }

      // set Y axis
      this.chartOptions.series = []
      for (const [name, userData] of this.trelloCards.entries()) {
        var serie = {}
        serie.name = name
        serie.data = []
        this.chartOptions.xAxis.categories.forEach(date => {
          if (userData.has(date)) {
            serie.data.push(userData.get(date).length)
          } else {
            serie.data.push(0)
          }
        })
        this.chartOptions.series.push(serie)
      }
    }
  }
}
</script>
