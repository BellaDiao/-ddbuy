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

      <div style="flex: 6">
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
          :min-date="minDate"
          :max-date="maxDate"
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
          style="width: 100%; padding: 4px 0"
          :row-style="{ height: '0' }"
          :cell-style="{ padding: '0' }"
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
          <el-table-column prop="finishR" label="完成率" width="82" sortable>
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
    // console.log(this.date, this.searchType)
    this.initData();
  },
  mounted () {
    this.$nextTick(function () {

    })
  },
  data () {
    return {
      data: '',
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
      mulPage: true,//分页
      currentPage: 1,
      pageSize:10,
      barData: {
        xData: '',
        yData1: '',
        yData2: '',
      },
      cmd: 'com.awspaas.user.apps.project.dynamics.getReportInfo',
      minDate: new Date(new Date().setFullYear(new Date().getFullYear() - 2)),
      maxDate: new Date(new Date().setFullYear(new Date().getFullYear() + 2))

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
      // 请求本地假数据：
      // axios
      //   .post('http://localhost:3001/homeNum')

      //   .then(response => {
      //     let dataT = response.data.data
      //     this.tableData = dataT.tableData
      //     this.finishNum = dataT.finishNum
      //     this.unfinishNum = dataT.unfinishNum
      //     this.totalhNum = dataT.totalhNum
      //     this.barData.xData = dataT.barData.xData
      //     this.barData.yData1 = dataT.barData.yData1
      //     this.barData.yData2 = dataT.barData.yData2
      //     this.finishChart();
      //     this.unfinishChart();
      //     this.barChart();
      //   })
      

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

      //  正式代码：
      
      var sid = this.$route.query.sid;// 获取sid

      axios.post('https://sast.awspaas.com/portal/r/w?sid=' + sid + '&cmd=' + this.cmd + '&searchType=' + this.searchType + '&beginDate=' + this.beginDate + '&endDate=' + this.endDate,
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
          // console.log("至今:", this.date);
          this.initData();
          break;
        case '当月':
          this.date = this.getMonthTime();
          // console.log("当月:", this.date);
          this.initData();
          break;
        case '当日':
          this.date = this.getDayTime();
          // console.log("当日:", this.date);
          this.initData();
          break;
        case '当年':
          this.date = this.getYearTime();
          // console.log("当年:", this.date);
          this.initData();
          break;
      }
      this.qukshow = false//关闭弹框
    },
    getDayTime () {      //获取当日
      this.searchType = 'nowDay';
      let date = new Date;
      let y = date.getFullYear()
      let m = date.getMonth() + 1
      m = m < 10 ? '0' + m : m
      let d = date.getDate()
      d = d < 10 ? ('0' + d) : d
      return `${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`;
    },
    getMonthTime () {      //获取当月
      this.searchType = 'nowMonth';
      let date = new Date;
      let y = date.getFullYear()
      let m = date.getMonth() + 1
      m = m < 10 ? '0' + m : m
      let d = date.getDate()
      d = d < 10 ? ('0' + d) : d
      return `${date.getFullYear()}-${date.getMonth() + 1}`;
    },
    getYearTime () {      //获取当年
      this.searchType = 'nowYear';
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
      this.searchType = 'annual';
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
      this.searchType = 'autoRule';
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
  outline: 1px solid red;
    font-size: 12px;

}
#home {
  background-color: '#f5f5f5';
  // padding-bottom: 3rem;

  .head {
    margin-top: -3rem;
    width: 100%;
    background-image: url('http://518taole.7-orange.cn/backImage.png');
  }
}

// .van-divider {
//   background-color: #f5f5f5;
//   margin: 0;
// }
//
// .top-wrapper {
//   width: 100%;
//   height: 200px;
//   background-color: rgb(17, 142, 234);
//   padding-top: 10px;
// }

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
// .el-table .el-table__cell {
//   // 间距
//   padding-top: 4px !important;
//   padding-bottom: 4px !important;
// }
thead{
 background-color: red !important;
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
  margin-top:3px;
  text-align: center;
  font-size: 13px !important;
}
.el-table th.el-table__cell > .cell {
  padding-left: 0px !important;
  padding-right: 0px !important;
}
.van-cell__value {
  text-align: left !important;
}
.el-table th.el-table__cell {
  font-size: 1.5rem !important;
  background-color: #c7e6f1 !important;
}
.el-table .cell {
  line-height: 16px !important;
}
.el-table th.el-table__cell>.cell{
    padding-left: 0px !important;
}

// .van-cell__title{
//   width: 100px  !important;
// }
</style>
