<!--
 * @Author: diaolihui
 * @Github: https://github.com/BellaDiao/-ddbuy
  * @LastEditTime :  
 * @Description: Home 首页模块
 * @FilePath: /ddBuy/src/views/home/Home.vue
 -->
<template>
  <div id="home">
    <div class="topContent" style="display: flex">
      <div style="flex: 1">
        <van-cell is-link @click="showPopup" v-model="carmodel">{{
          carmodel
        }}</van-cell>
        <van-popup
          v-model="qukshow"
          position="bottom"
          :style="{ height: '50%' }"
        >
          <van-area
            :area-list="areaList"
            ref="myArea"
            title="请选择"
            @change="timeFun"
            @confirm="onConfirm1"
            @cancel="onCancel1"
          />
        </van-popup>
      </div>

      <div style="flex: 4">
        <van-cell
          title="选择日期范围"
          :value="date"
          @click="show = true"
          class="title"
        />
        <van-calendar
          class="timing"
          v-model="show"
          type="range"
          @confirm="onConfirm"
          :show-confirm="false"
        />
        >
      </div>
    </div>
    <div class="mainContent1" style="display: flex">
      <div style="flex: 2" class="groove">
        <div class="zjh">总计划</div>
        <div class="number">{{ totalhNum }}</div>
      </div>

      <div style="flex: 2.5">
        <div style="float: left">
          <div class="littleT">已完成</div>
          <div ref="finishChart" style="width: 120px; height: 120px"></div>
        </div>
      </div>
      <div style="flex: 2.5">
        <div style="float: left">
          <div class="littleT">未完成</div>
          <div ref="unfinishChart" style="width: 120px; height: 120px"></div>
        </div>
      </div>
    </div>

    <div class="mainContent2">
      <div ref="barChart" style="height: 150px"></div>
    </div>
    <div class="bottomContent">
      <div class="littleT">员工计划完成情况</div>
      <div class="hello">
        <el-table
          :data="
            tableData.slice(
              (currentPage - 1) * pageSize,
              currentPage * pageSize
            )
          "
          tooltip-effect="dark"
          style="width: 100%"
        >
          <el-table-column type="index" width="50" label="序号" align="center">
            <template slot-scope="scope">
              <span>{{ (currentPage - 1) * pageSize + scope.$index + 1 }}</span>
            </template>
          </el-table-column>

          <el-table-column prop="dept" label="部门" width="100">
          </el-table-column>
          <el-table-column prop="psn" label="责任人/分配人" width="65">
          </el-table-column>
          <el-table-column prop="finish" label="已完成" width="50">
          </el-table-column>
          <el-table-column prop="unfinish" label="未完成" width="50">
          </el-table-column>
          <el-table-column prop="finishR" label="完成率" width="80" sortable>
          </el-table-column>
        </el-table>

        <el-pagination
          :page-size="pageSize"
          :current-page="currentPage"
          @current-change="handleCurrentChang"
          layout="prev, pager, next, total"
          :total="tableData.length"
        >
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script type="text/javascript">

// 引入Vuex
import { mapMutations, mapState } from 'vuex'

import $ from 'jquery';
// 引入页面组件
import Vue from 'vue';
import { Area, Popup } from 'vant';
import { Calendar } from 'vant';
import { Cell, CellGroup } from 'vant';
import * as echarts from 'echarts';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

import { getHomeData } from './../../serve/api/home.js'
import axios from 'axios'

import qs from 'qs'

Vue.use(Cell);
Vue.use(CellGroup);
Vue.use(Area);
Vue.use(Popup);

Vue.use(Calendar);
Vue.use(ElementUI);
const optionsItem = ['50', '100'];

export default {

  computed: {
    ...mapState(['userInfo']),
  },
  // beforeCreate () {
  //   axios
  //     .post('http://localhost:3001/home')
  //     .then(response => {
  //       console.log("beforeCreate:",response.data)
  //       this.tableData = response.data
  //     })
  // },
  created () {
    // 0.数据初始化
    this.date = this.getTodayTime();//默认查询至今
    this.initData();
  },
  mounted () {
    this.$nextTick(function () {

    })
  },
  data () {
    return {
      date: '',//时间范围
      show: false,//时间范围控件是否显示
      qukshow: false,//快捷时间查询弹框是否显示
      carmodel: '至今',
      searchType: '',
      beginDate: '',
      endDate: '',
      areaList: {
        province_list: {
          110000: '至今',
          120000: '当月',
          130000: '当日',
          140000: '当年',
        }
      },
      finishNum: '',
      unfinishNum: '',
      totalhNum: '',
      tableData: '',
      // tableData: [{
      //   xh: '1',
      //   dept: '信息中心',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "12%",
      // }, {
      //   xh: '2',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "1%",

      // }, {
      //   xh: '3',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "42%",
      // }, {
      //   xh: '4',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "33%",
      // }, {
      //   xh: '5',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "42%",

      // }, {
      //   xh: '6',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "42%",

      // }, {
      //   xh: '7',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "28%",

      // }, {
      //   xh: '8',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "26%",

      // }, {
      //   xh: '9',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "93%",
      // }, {
      //   xh: '10',
      //   dept: '信息中心',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "62%",
      // }, {
      //   xh: '12',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "41%",

      // }, {
      //   xh: '13',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "48%",
      // }, {
      //   xh: '14',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "33%",
      // }, {
      //   xh: '15',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "82%",

      // }, {
      //   xh: '16',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "32%",

      // }, {
      //   xh: '17',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "78%",

      // }, {
      //   xh: '18',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "19%",

      // }, {
      //   xh: '19',
      //   dept: '上海市普陀区',
      //   psn: '王小虎',
      //   finish: 12,
      //   unfinish: 12,
      //   finishR: "73%",
      // },],
      mulPage: true,//分页
      currentPage: 1,
      pageSize: 6,
      barData: {
        xData: '',
        yData1: '',
        yData2: '',
      },
    }
  },
  // components: {
  //   Header,
  //   Sowing,
  //   Tip,
  //   Nav,
  //   VipTip,
  //   FlashBuy,
  //   SpecialZone,
  //   TabbarGoodsItem,
  //   Loading,
  // },
  methods: {

    // Vuex中的方法
    ...mapMutations(['ADD_GOODS', 'ADD_TO_CART']),
    // 数据初始化
    initData () {

      /**
       * 请求数据
       * */

      // axios
      //   .post('http://localhost:3001/homeNum')

      //   .then(response => {
      //     let dataT = response.data.data
      //     this.tableData = dataT.tableData
      //      this.finishNum = dataT.finishNum
      //     this.unfinishNum = dataT.unfinishNum
      //     this.totalhNum = dataT.totalhNum
      //     this.barData.xData = dataT.barData.xData
      //     this.barData.yData1 = dataT.barData.yData1
      //     this.barData.yData2 = dataT.barData.yData2
      //     this.finishChart();
      //     this.unfinishChart();
      //     this.barChart();

      //   })
      // 


      // axios.post('/bpm/portal/r/w',{
      //     searchType: 'nowYear',
      //     beginDate: '2021-01-01',
      //     endDate: '2021-10-14',
      //     sid: '3fec0ea5-db8b-432f-a0cd-c1123038adef',
      //     cmd: 'com.awspaas.user.apps.project.dynamics.getReportInfo'
      //   },
      //   {contentType: 'application/json',}
      // ).then(response => {
      //   console.log("shuju:", response)

      // })

      axios.post('/bpm/portal/r/w?sid=3fec0ea5-db8b-432f-a0cd-c1123038adef&cmd=com.awspaas.user.apps.project.dynamics.getReportInfo&searchType=nowYear&beginDate=2021-01-01&endDate=2021-10-14',
      ).then(response => {

        let dataT = response.data.data
        this.tableData = dataT.tableData
        this.finishNum = dataT.finishNum
        this.unfinishNum = dataT.unfinishNum
        this.totalhNum = dataT.totalhNum
        this.barData.xData = dataT.barData.xData
        this.barData.yData1 = dataT.barData.yData1
        this.barData.yData2 = dataT.barData.yData2
        this.finishChart();
        this.unfinishChart();
        this.barChart();

      })
    },

    // ******
    //获取表格序号
    getIndex ($index) {
      return (this.currentPage - 1) * this.pageSize + $index + 1
    },
    // 分页
    handleCurrentChang (currentPage) {
      this.currentPage = currentPage
    },

    // *****
    // 快捷时间查询开始
    showPopup () {
      this.qukshow = true;
    },

    onConfirm1 (val) { //确定选择
      // console.log(val[0].name)
      this.carmodel = val[0].name;

      switch (this.carmodel) {
        case '至今':
          this.date = this.getTodayTime();
          this.searchType = 'annual';

          console.log("至今:", this.date);
          this.initData();
          break;
        case '当月':
          this.date = this.getMonthTime();
          this.searchType = 'nowMonth';

          console.log("当月:", this.date);
          this.initData();
          break;
        case '当日':
          this.date = this.getDayTime();
          this.searchType = 'nowDay';

          console.log("当日:", this.date);
          this.initData();
          break;
        case '当年':
          this.date = this.getYearTime();
          this.searchType = 'nowYear';
          console.log("当年:", this.date);
          this.initData();
          break;

      }
      this.qukshow = false//关闭弹框
    },
    getDayTime () {      //获取当日
      let date = new Date;
      let y = date.getFullYear()
      let m = date.getMonth() + 1
      m = m < 10 ? '0' + m : m
      let d = date.getDate()
      d = d < 10 ? ('0' + d) : d
      return `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`;
    },
    getMonthTime () {      //获取当月
      let date = new Date;
      let y = date.getFullYear()
      let m = date.getMonth() + 1
      m = m < 10 ? '0' + m : m
      let d = date.getDate()
      d = d < 10 ? ('0' + d) : d
      return `${date.getFullYear()}-${date.getMonth() + 1}`;
    },
    getYearTime () {      //获取当年
      let date = new Date;
      let y = date.getFullYear()
      let m = date.getMonth() + 1
      m = m < 10 ? '0' + m : m
      let d = date.getDate()
      d = d < 10 ? ('0' + d) : d
      // return `${date.getFullYear()}-${date.getFullYear()}`;
      return `${date.getFullYear()}`;
    },
    getTodayTime () {      //获取至今
      let date = new Date;
      let y = date.getFullYear()
      let m = date.getMonth() + 1
      m = m < 10 ? '0' + m : m
      let d = date.getDate()
      d = d < 10 ? ('0' + d) : d
      // return `${date.getFullYear()}/1/1-${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;
      return `初始日期-${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`;

    },

    onCancel1 () {//取消选中
      this.show = false
      this.$refs.myArea.reset()// 重置列表
    },
    // 
    timeFun (val) {
      console.log("快捷时间选择")
    },
    // 日期控件开始
    formatDate (date) {
      return `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`;
    },
    onConfirm (date) {
      const [start, end] = date;
      this.show = false;
      this.date = `${this.formatDate(start)}~${this.formatDate(end)}`;
      this.carmodel = '查询';
      this.searchType = 'annual';
      this.beginDate = this.formatDate(start)
      this.endDate = this.formatDate(end)

      this.initData();
    },
    finishChart () {    // 完成数量图
      var that = this;

      let value = that.finishNum;
      let maxValue = that.totalhNum;

      let pipe = echarts.init(this.$refs.finishChart);
      let option = {
        title: {
          textStyle: {
            fontWeight: 'bold',
            fontFamily: 'Microsoft YaHei',
            fontSize: 20
          },
          //标题位置
          top: '5%',
          left: '5%'
        },
        //环形中间文字
        graphic: [
          //第一行文字
          //内容 + 位置
          {
            type: 'text',
            left: 'center',
            top: '48%',
            style: {
              //value当前进度
              text: '' + value + '个',
              textAlign: 'center',
              fill: '#000',
              fontSize: 20
            }
          },
          //第二行文字
          //内容 + 位置
          // {
          //   type: 'text',
          //   left: 'center',
          //   top: '55%',
          //   style: {
          //      text: '共' + maxValue + '个',
          //     textAlign: 'center',
          //     fill: '#999',
          //     fontSize: 24
          //   }
          // }
        ],
        series: [{
          type: 'pie',
          radius: ['70%', '54%'],//['外圆大小', '内圆大小']
          center: ['50%', '50%'],//圆心位置['左右'， '上下']
          hoverAnimation: false,//取消鼠标悬停动画
          animationEasing: 'cubicOut',//设置动画缓动效果
          //取消显示饼图自带数据线条
          labelLine: {
            normal: {
              show: false
            }
          },
          //增加阴影效果
          itemStyle: {
            normal: {
              shadowBlur: 200,
              shadowColor: 'rgba(44, 196, 196, 0.8)'
            }
          },
          data: [
            //value当前进度 + 颜色
            {
              value: value,
              itemStyle: {
                normal: {
                  color: '#73A0FA'
                }
              }
            },
            //(maxValue进度条最大值 - value当前进度) + 颜色
            {
              value: maxValue - value,
              itemStyle: {
                normal: {
                  color: 'white'
                }
              }
            }
          ]
        }]
      };
      pipe.setOption(option);
      //随着浏览器窗口大小改变而改变
      window.addEventListener("resize", function () {
        pipe.resize();
      });
    },
    unfinishChart () {// 未完成数量图
      var that = this;
      let value = that.unfinishNum;
      let maxValue = that.totalhNum;
      let pipe = echarts.init(this.$refs.unfinishChart);
      let option = {
        title: {
          textStyle: {
            fontWeight: 'bold',
            fontFamily: 'Microsoft YaHei',
            fontSize: 20
          },
          //标题位置
          top: '5%',
          left: '5%'
        },
        //环形中间文字
        graphic: [
          //第一行文字
          //内容 + 位置
          {
            type: 'text',
            left: 'center',
            top: '48%',
            style: {
              //value当前进度
              text: '' + value + '个',
              textAlign: 'center',
              fill: '#000',
              fontSize: 20
            }
          },
          //第二行文字
          //内容 + 位置
          // {
          //   type: 'text',
          //   left: 'center',
          //   top: '55%',
          //   style: {
          //      text: '共' + maxValue + '个',
          //     textAlign: 'center',
          //     fill: '#999',
          //     fontSize: 24
          //   }
          // }
        ],
        series: [{
          type: 'pie',
          radius: ['70%', '54%'],//['外圆大小', '内圆大小']
          center: ['50%', '50%'],//圆心位置['左右'， '上下']
          hoverAnimation: false,//取消鼠标悬停动画
          animationEasing: 'cubicOut',//设置动画缓动效果
          //取消显示饼图自带数据线条
          labelLine: {
            normal: {
              show: false
            }
          },
          //增加阴影效果
          itemStyle: {
            normal: {
              shadowBlur: 200,
              shadowColor: 'rgba(44, 196, 196, 0.8)'
            }
          },
          data: [
            //value当前进度 + 颜色
            {
              value: value,
              itemStyle: {
                normal: {
                  color: 'red'
                }
              }
            },
            //(maxValue进度条最大值 - value当前进度) + 颜色
            {
              value: maxValue - value,
              itemStyle: {
                normal: {
                  color: 'white'
                }
              }
            }
          ]
        }]
      };
      pipe.setOption(option);
      //随着浏览器窗口大小改变而改变
      window.addEventListener("resize", function () {
        pipe.resize();
      });
    },
    barChart () { // 柱状图
      let barChart = echarts.init(this.$refs.barChart);

      let option = {
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        legend: {},
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          data: this.barData.xData
        },
        yAxis: {
          type: 'value',
        },
        series: [
          {
            name: '已完成',
            type: 'bar',
            stack: 'total',
            barWidth: 30,
            label: {
              show: true
            },
            emphasis: {
              focus: 'series'
            },
            data: this.barData.yData1
          },
          {
            name: '未完成',
            type: 'bar',
            stack: 'total',
            label: {
              show: true
            },
            emphasis: {
              focus: 'series'
            },
            data: this.barData.yData2
          },

        ]
      };

      option && barChart.setOption(option);
    },
  },
}
</script>

<style lang="less" scoped>
html * {
  font-size: 8px;
  // outline: 1px solid red;
}
#home {
  background-color: '#f5f5f5';
  padding-bottom: 3rem;

  .head {
    margin-top: -3rem;
    width: 100%;
    background-image: url('http://518taole.7-orange.cn/backImage.png');
  }
}
.van-divider {
  background-color: #f5f5f5;
  margin: 0;
}
//
.top-wrapper {
  width: 100%;
  height: 200px;
  background-color: rgb(17, 142, 234);
  padding-top: 10px;
}

.groove {
  border-style: groove;
  height: 138px;
  width: 150px;
  background-color: #e4f4f4;
}
.topContent {
  height: 2rem;
}
.mainContent1 {
  height: 8rem;
}
.mainContent2 {
  height: 9.5rem;
  width: 100%;
}
.bottomContent {
  height: 8rem;
}
.el-table .el-table__cell {
  padding: 4px 0 !important;
}
.van-cell {
  padding: 5px 3px !important;
}
.zjh,
.number {
  text-align: center;
  margin-top: 28px;
  font-size: 24px !important;
}
.littleT {
  text-align: center;
  font-size: 16px !important;
}
.el-table th.el-table__cell > .cell {
  padding-left: 0px !important;
  padding-right: 0px !important;
}
.van-cell__value {
  text-align: left !important;
}
// .timing{

// }
</style>
