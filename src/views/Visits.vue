<template>
  <div>
    <ul class='search-panel'>
      <li>
        <div>
          <span>时间</span>
          <Radio-group v-model='currentDateType' type='button' class='search-date-type'>
            <Radio v-for='item in DATETYPES' :label='item.value' :key='item'>{{ item.text }}</Radio>
          </Radio-group>
          <Date-picker v-model='currentDate' type='daterange' placeholder='选择日期' class='search-date' :editable='false' @on-open-change='currentDateType=-1'></Date-picker>
        </div>
      </li>
    </ul>
    <charts id='project-chart' class='visit-chart' :options="projectChartOptions" :loading="projectLoading" ref="projectChart" @click="clickProject"></charts>
    <charts v-if="showModuleChart" id='module-chart' class='visit-chart' :options="moduleChartOptions" :loading="moduleLoading" ref="moduleChart"></charts>
  </div>
</template>

<script>
import Charts from '@/components/Charts'
import { LayoutMixin, ChartMixin } from '@/mixins'
import { GET_VISITS } from '@/store/types'
const DATETYPES = [
  {
    text: '最近一周',
    value: 0
  },
  {
    text: '最近一月',
    value: 1
  },
  {
    text: '自定义',
    value: -1
  }
]
const barStyle = {
  type: 'bar',
  barMaxWidth: 40,
  itemStyle: {
    normal: {
      barBorderRadius: [5, 5, 0, 0]
    }
  }
}
const projectNames = ['SuperIT', 'SuperEnvMall', 'SuperLAB']
export default {
  mixins: [LayoutMixin, ChartMixin],
  components: { Charts },
  data () {
    return {
      activeTopMenu: 'home',
      openLeftMenus: ['operations'],
      currentDateType: -1,
      currentDate: ['2017-06-01', '2017-06-03'],
      DATETYPES,
      showModuleChart: false
    }
  },
  computed: {
    projectChartOptions () {
      const series = []
      const { xData, yData } = this.$store.state.Visit.projectData
      for (let name of projectNames) {
        series.push({
          name: name,
          ...barStyle,
          data: yData[name]
        })
      }
      const totalSerie = {
        type: 'line',
        name: '总人数',
        data: []
      }
      for (let i in xData) {
        let sum = 0
        for (let oSerie of series) {
          sum += oSerie.data[i]
        }
        totalSerie.data.push(sum)
      }
      series.push(totalSerie)
      return {
        legend: {
          data: [...projectNames, totalSerie.name]
        },
        xAxis: {
          data: xData
        },
        yAxis: {
          type: 'value',
          name: '人数'
        },
        series
      }
    },
    projectLoading () {
      return this.$store.state.Visit.projectLoading
    },
    moduleChartOptions () {
      const { xData, yData } = this.$store.state.Visit.moduleData
      const moduleQuery = this.$store.state.Visit.moduleQuery
      return {
        title: {
          text: `日期：${moduleQuery.date} 项目：${moduleQuery.project || '全部'}`,
          textStyle: {
            fontSize: 12
          },
          left: 'center'
        },
        xAxis: {
          data: xData
        },
        yAxis: {
          type: 'value',
          name: '人数'
        },
        series: [{
          name: '人数',
          ...barStyle,
          data: yData
        }]
      }
    },
    moduleLoading () {
      return this.$store.state.Visit.moduleLoading
    }
  },
  watch: {
    currentDate () {
      this.getProjectVistis()
    },
    currentDateType () {
      let [start, end] = this.currentDate
      const fmt = 'YYYY-MM-DD'
      const now = this.$moment().format(fmt)
      switch (this.currentDateType) {
        case 0:
          end = now
          start = this.$moment().subtract(6, 'days').format(fmt)
          break
        case 1:
          end = now
          start = this.$moment().subtract(1, 'months').format(fmt)
          break
      }
      this.currentDate = [start, end]
    }
  },
  created () {
    this.getProjectVistis()
  },
  methods: {
    getProjectVistis () {
      this.showModuleChart = false
      this.$store.dispatch(GET_VISITS, {
        projectLoading: true,
        projectQuery: {
          names: projectNames,
          date: this.currentDate
        },
        moduleQuery: {}
      })
    },
    clickProject ({ seriesName, name }) {
      this.showModuleChart = true
      this.$store.dispatch(GET_VISITS, {
        moduleLoading: true,
        moduleQuery: {
          project: seriesName === '总人数' ? null : seriesName,
          date: name
        }
      })
    }
  }
}
</script>

<style lang='less' scoped>
.visit-chart {
  height: 300px;
  padding: 10px 0;
}
</style>
