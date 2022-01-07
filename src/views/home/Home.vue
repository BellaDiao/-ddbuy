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
      <div style="flex: 3">
        <van-dropdown-menu style="font-size: 10px; height: 24px">
          <van-dropdown-item
            v-model="value1"
            :options="deptOption"
            @change="deptChange()"
          />
        </van-dropdown-menu>
      </div>
      <div class="time" style="flex: 4; height: 48px">
        <van-cell title="" :value="date" @click="show = false" class="title" />
        <van-calendar
          class="timing"
          v-model="show"
          type="range"
          @confirm="onConfirm"
          :show-confirm="false"
          :min-date="minDate"
          :max-date="maxDate"
        />
      </div>
      <div style="flex: 2" class="">
        <van-cell is-link @click="showPopup" v-model="carmodel" class="quick">{{
          carmodel
        }}</van-cell>
        <van-popup
          class=""
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
    </div>
    <div class="mainContent1" style="display: flex; margin-top: 16px">
      <div style="flex: 2" class="groove">
        <div class="zjh" style="margin-top: 19px">项目</div>
        <div class="zjh">总数</div>
        <div class="number">{{ totalhNum }}</div>
      </div>

      <div style="flex: 2.5">
        <div style="float: left" class="groove">
          <div class="littleT"><strong>已完成√</strong></div>
          <div
            ref="finishChart"
            style="width: 120px; height: 120px; margin-left: 16px"
          ></div>
        </div>
      </div>
      <div style="flex: 2.5">
        <div style="float: left" class="groove">
          <div class="littleT"><strong>未完成×</strong></div>
          <div
            ref="unfinishChart"
            style="width: 120px; height: 120px; margin-left: 16px"
          ></div>
        </div>
      </div>
    </div>
    <div class="mainContent2">
      <div ref="barChart" style="height: 150px"></div>
    </div>
    <div class="bottomContent">
      <van-divider
        :style="{ color: '#1989fa', borderColor: '#1989fa', padding: '0 16px' }"
      >
        <span style="font-size: 16px; font-weight: 300">员工计划完成情况</span>
      </van-divider>
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
          v-on:sort-change="sortChange"
        >
          <el-table-column type="index" width="40" label="序号" align="center">
            <template slot-scope="scope">
              <span>{{ (currentPage - 1) * pageSize + scope.$index + 1 }}</span>
            </template>
          </el-table-column>

          <el-table-column prop="dept" label="部门" width="100">
          </el-table-column>
          <el-table-column prop="psn" label="负责人/分配人" width="65">
          </el-table-column>
          <el-table-column prop="finish" label="√" width="50">
          </el-table-column>
          <el-table-column prop="unfinish" label="×" width="50">
          </el-table-column>
          <el-table-column
            prop="finishR"
            label="完成率"
            width="82"
            sortable="custom"
          >
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
import { DropdownMenu, DropdownItem } from 'vant';

import * as echarts from 'echarts';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

import axios from 'axios'


Vue.use(Cell);
Vue.use(CellGroup);
Vue.use(Area);
Vue.use(Popup);

Vue.use(Calendar);
Vue.use(DropdownMenu);
Vue.use(DropdownItem);
Vue.use(ElementUI);
const optionsItem = ['50', '100'];

export default {

  computed: {
    ...mapState(['userInfo']),
  },

  created () {
    // 0.数据初始化
    this.date = this.getMonthTime();//默认查询至今
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
      // readonly: true,
      qukshow: false,//快捷时间查询弹框是否显示
      carmodel: '查询',
      searchType: '',
      beginDate: '',
      endDate: '',
      areaList: {
        province_list: {
          110000: '至今',
          120000: '当月',
          130000: '当日',
          140000: '当年',
          150000: '自定义时间',

        }
      },
      finishNum: '',
      unfinishNum: '',
      totalhNum: '',
      tableData: '',
      mulPage: true,//分页
      currentPage: 1,
      pageSize: 10,
      barData: {
        xData: '',
        yData1: '',
        yData2: '',
      },
      cmd: 'com.awspaas.user.apps.project.dynamics.getReportInfo',
      minDate: new Date(new Date().setFullYear(new Date().getFullYear() - 2)),
      maxDate: new Date(new Date().setFullYear(new Date().getFullYear() + 2)),
      sortType: '',
      data: "",
      value1: "0",
      deptOption: [
      //   {
      //     text: "所有部门",
      //     value: "0"
      //   }, {
      //     text: "有效载荷事业部",
      //     value: "a38ccaaf-d1dc-4366-8c81-9b7a804ea122"
      //   }, {
      //     text: "有效载荷系统室",
      //     value: "91014634-9a5e-40fc-b84b-d231efb86fb9"
      //   }, {
      //     text: "有效载荷专业室",
      //     value: "f72c6e65-e22b-4542-a538-0ae6c34e47a1"
      //   }, {
      //     text: "运载事业部",
      //     value: "b820bcfd-133f-47ee-8692-a066e86e041e"
      //   }, {
      //     text: "测发控系统室",
      //     value: "4dd65826-17ba-4806-ba8a-75fc7282c198"
      //   }, {
      //     text: "箭载电子室",
      //     value: "604ca942-0530-4e0b-8b60-9f9a283d8ff1"
      //   }
      ]

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
       * 请求本地假数据：
       * */

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


      //  正式代码：
      // https://sast.awspaas.com/portal
      var sid = this.$route.query.sid;// 获取sid

      if (this.value1 == "0" || this.value1 == 0) {
        axios.post(window.location.href.substring(0, window.location.href.indexOf("portal/")) + 'portal/r/w?sid=' + sid +
          '&cmd=' + this.cmd + '&searchType=' + this.searchType +
          '&beginDate=' + this.beginDate + '&endDate=' + this.endDate + '&sortType=' + this.sortType + '&queryDeptId=',
        ).then(response => {
          let dataT = response.data.data

          this.deptOption = dataT.deptList

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
      } else {
        axios.post(window.location.href.substring(0, window.location.href.indexOf("portal/")) + 'portal/r/w?sid=' + sid +
          '&cmd=' + this.cmd + '&searchType=' + this.searchType +
          '&beginDate=' + this.beginDate + '&endDate=' + this.endDate + '&sortType=' + this.sortType + '&queryDeptId=' + this.value1,
        ).then(response => {
          let dataT = response.data.data

          this.deptOption = dataT.deptList

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
      }



      /**
       
       "deptList": [{
         "deptName": "有效载荷事业部",
         "deptId": "a38ccaaf-d1dc-4366-8c81-9b7a804ea122"
        }, {
         "deptName": "有效载荷系统室",
         "deptId": "91014634-9a5e-40fc-b84b-d231efb86fb9"
        }, {
         "deptName": "有效载荷专业室",
         "deptId": "f72c6e65-e22b-4542-a538-0ae6c34e47a1"
        }, {
         "deptName": "运载事业部",
         "deptId": "b820bcfd-133f-47ee-8692-a066e86e041e"
        }, {
         "deptName": "测发控系统室",
         "deptId": "4dd65826-17ba-4806-ba8a-75fc7282c198"
        }, {
         "deptName": "箭载电子室",
         "deptId": "604ca942-0530-4e0b-8b60-9f9a283d8ff1"
        }],
      
        // var url = window.location.href.substring(0,window.location.href.indexOf("portal/"))+"portal/r/w?sid=" + sid + 
      // "&cmd=CLIENT_DW_FORM_MODIFYPAGE&bindid=" + bindId + "&processDefId=obj_a98ce763840e4e25a3b5dbbd17c0753a&taskInstId=0";
      // https://sast.awspaas.com/portal/apps/com.awspaas.user.apps.project.dynamics/kb/index.html#/dashboard/home?sid=62d05599-5f8f-4c4d-8e0e-cea541acd9b4

      
       * **/
    },
    deptChange () {
      // var object =  { text: '全部1', value: 10 }
      // this.deptOption.unshift(object);
      // alert(this.deptOption)
      // console.log(this.value1)

    },
    // table排序
    sortChange ({ prop, order }) {
      console.log("order:", order)
      if (order == 'ascending') {
        this.sortType = 'asc';
        // console.log("this.sortType:",this.sortType)
      }
      if (order == 'descending') {
        this.sortType = 'desc';
        // console.log("this.sortType:",this.sortType)
      }
      if (order == '') {
        this.sortType = '';
        // console.log("this.sortType:",this.sortType)
      }
      this.initData();
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
          this.initData();
          break;
        case '当月':
          this.date = this.getMonthTime();
          this.initData();
          break;
        case '当日':
          this.date = this.getDayTime();
          // console.log("当日:", this.date);
          this.initData();
          break;
        case '当年':
          this.date = this.getYearTime();
          this.initData();
          break;
        case '自定义时间':
          this.show = true;
          // this.date = this.getYearTime();
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
      return `起始~${date.getFullYear()}-${date.getMonth() + 1}-${date.getDate()}`;

    },

    onCancel1 () {//取消选中
      this.show = false
      this.$refs.myArea.reset()// 重置列表
    },
    // 
    timeFun (val) {
      console.log("查询")
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
          {
            type: 'text',
            left: 'center',
            top: '48%',
            style: {
              text: '' + value + '个',
              textAlign: 'center',
              fill: '#000',
              fontSize: 20
            }
          },

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
              shadowColor: '#e4f4f4'
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
              // shadowColor: 'rgba(44, 196, 196, 0.8)'
              shadowColor: '#e4f4f4'

            }
          },
          data: [
            //value当前进度 + 颜色
            {
              value: value,
              itemStyle: {
                normal: {
                  color: '#ff0000'
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
          height: 130,
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          data: this.barData.xData,
          axisLabel: { interval: 0 }
          // data: ["有效载荷系统室", "有效载荷事业部", "有效载荷专业室"]
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
              normal: {
                show: true,
                // formatter: function (params) {// 值为0的时候不在柱子上显示
                //   if (params.value > 0) {
                //     return params.value;
                //   } else {
                //     return '';
                //   }
                // },
              }
            },
            // 柱子颜色
            itemStyle: {
              normal: {
                color: '#73A0FA'
              }
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
              normal: {
                show: true,
                // formatter: function (params) {
                //   if (params.value > 0) {
                //     return params.value;
                //   } else {
                //     return '';
                //   }
                // },
              }

            },
            itemStyle: {
              normal: {
                color: '#ff0000'
              }
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
  // outline: 1px solid red;
  // font-size: 13px;
}
// .topContent :first-child {
//   height: 40px !important;
// }

// .topContent .van-dropdown-menu .van-dropdown-menu__bar{
//     height: 35px !important;
// }
.van-cell {
  font-size: 10px !important;
}
.el-table
  .el-table--fit
  .el-table--scrollable-x
  .el-table--enable-row-hover
  .el-table--enable-row-transition {
  font-size: 15px !important;
}
.timing {
  font-size: 10px !important;
}
.el-table__header .el-table thead {
  color: #303133 !important;
  font-weight: 500;
}
.el-table__header {
  font-weight: 500;
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
  // margin-top: 25 px;
}
.mainContent2 {
  height: 9rem;
  width: 100%;
}
.bottomContent {
  height: 8rem;
}

.van-cell {
  padding: 5px 3px !important;
}
.zjh {
  text-align: center;
  // margin-top: 19px;
  font-size: 22px !important;
}
.number {
  text-align: center;
  margin-top: 19px;
  font-size: 22px !important;
}
.littleT {
  margin-top: 10px;
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
.el-table__header-wrapper thead .el-table .cell {
  line-height: 16px !important;
}
.el-table th.el-table__cell > .cell {
  padding-left: 0px !important;
}

.van-divider {
  margin: 0 0 !important;
}

.el-table__header-wrapper thead .el-table th.el-table__cell > .cell {
  color: black !important;
  padding-left: 0px !important;
  padding-right: 0px !important;
}
.quick {
  border: 1px solid white !important;
  border-radius: 5px;
  font-size: 14px !important;
  color: brown !important;
  background-color: rgb(17, 164, 223) !important;
}
// 日历
.van-cell__title {
  -webkit-box-flex: 1;
  -webkit-flex: 1;
  flex: 0.1 !important;
}
</style>
