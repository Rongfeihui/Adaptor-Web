<template>
  <div class="dashboard">
    <el-row>
      <el-col :span="24">
        <div id="gotobedbar"></div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import echarts from 'echarts';
  import macarons from 'echarts/theme/macarons';
  import data from '../../../static/data/data.json';
  import {mapGetters} from 'vuex'
  export default {
    data() {
      return {
        chart: null,
      };
    },
    props:{
      historyData:{
        type: Object,
        default () {
          return {}
        }
      },
    },
    computed: {
      ...mapGetters({
        sidebar: 'sidebar',
        device: 'device',
      }),
    },
    methods: {
      drawbar(id) {
        let o = document.getElementById(id);
        let height = document.documentElement.clientHeight;
        height =height*0.75 ;
        /*o.style.height= height+"px";*/
        this.chart = echarts.init(o,'macarons');
        this.chart.setOption(option,true);
      }
    },
    mounted() {
      this.$nextTick(function () {
        this.drawbar('gotobedbar');
        let that = this;
        let resizeTimer = null;
        window.onresize = function () {
          if (resizeTimer) clearTimeout(resizeTimer);
          resizeTimer = setTimeout(function () {
            that.drawbar('gotobedbar');
          }, 300);
        }
      });
    },
    watch: {
      historyData: {
        handler(newValue, oldValue) {
          let series=[]
          let data8=[]
          let data16=[]
          let legendData=[]
          let yAxis=[]
          if(JSON.stringify(newValue)!= "{}") {
            let observations = newValue.observations
            for (let i = 0; i < observations.length; i++) {
              if (observations[i].hour == 8)
                data8.push(observations[i].obsValue)
              else data16.push(observations[i].obsValue)
            }
            series.push({
                name: "8:00",
                type: 'line',
                tiled: '总量',
                areaStyle: {normal: {}},
                data: data8
              },
            )
            series.push({
                name: "18:00",
                type: 'line',
                tiled: '总量',
                areaStyle: {normal: {}},
                data: data16
              },
            )
            yAxis.push({type:"value",
              axisLabel:{
                formatter:"{value} "+newValue.uom
              }})
            legendData.push("8:00","18:00")
            option.series=series
            option.legend.data=legendData
            option.title.text=newValue.sensorType+"数据月统计"
            option.yAxis=yAxis
            this.drawbar('gotobedbar')
          }
        },
        deep: true
      },
      'sidebar.collapsed': function (val) {
        this.chart = {}
        let that = this;
        setTimeout(function () {
          that.drawbar('gotobedbar');
        }, 300);
      },
    }
  }
  const getBeforeDate = (n) => {
    let list = [];
    let d = new Date(); // 这个算法能自动处理闰年和非闰年。2012年是闰年，所以2月有29号
    let s = '';
    let i = 0;
    while (i < n) {
      d.setDate(d.getDate() - 1);
      let year = d.getFullYear();
      let mon = d.getMonth() + 1;
      let day = d.getDate();
      let date=year + "-" + (mon < 10 ? ('0' + mon) : mon) + "-" + (day < 10 ? ('0' + day) : day);
      list.push(date);
      i++;
    }
    return list.reverse();
  }
  const option = {
    title: {
      subtext: '从当天开始往前',
      left:'center',
    },
    tooltip: {
      trigger: 'axis',
      axisPointer : {            // 坐标轴指示器，坐标轴触发有效
        type : 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
      }
    },
    legend: {
      data: [],
      orient:'vertical',
      left:'right',
      top:'middle',//如果 top 的值为'top', 'middle', 'bottom'，组件会根据相应的位置自动对齐。
      itemGap:20
    },
    toolbox: {
      show: true,
      orient: 'horizontal',      // 布局方式，默认为水平布局，可选为：
      // 'horizontal' ¦ 'vertical'
      x: 'right',                // 水平安放位置，默认为全图右对齐，可选为：
                                 // 'center' ¦ 'left' ¦ 'right'
                                 // ¦ {number}（x坐标，单位px）
      y: 'top',                  // 垂直安放位置，默认为全图顶端，可选为：
                                 // 'top' ¦ 'bottom' ¦ 'center'
                                 // ¦ {number}（y坐标，单位px）
      color: ['#1e90ff', '#22bb22', '#4b0082', '#d2691e'],
      feature: {
        mark: {show: true},
        dataView: {show: true, readOnly: false},
        magicType: {show: true, type: ['line', 'bar', 'stack', 'tiled']},
        restore: {show: true},
        saveAsImage: {show: true}
      }
    },
    calculable: true,
    dataZoom: {
      show: true,
      realtime: true,
      start: 0,
      end: 100
    },
    xAxis: [
      {
        type: 'category',
        boundaryGap: false,
        data: getBeforeDate(30)
      }
    ],
    yAxis: [
      {
        type: 'value',
        axisLabel:{
          formatter:{}
        }
      }
    ],
    series:[]

  };

</script>

<style scoped>
  #gotobedbar {
    width: 100%;
    min-height: 350px;
    margin-right: 15px;
  }
</style>
