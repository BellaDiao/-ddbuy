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
      <!-- <van-area title="" :area-list="areaList" @change="timeFun" /> -->
      <div style="flex: 5">
      <div>快捷时间查询</div>
      <van-radio-group
        v-model="radio"
        direction="horizontal"
        
      >
        <van-radio name="1">至今</van-radio>
        <van-radio name="2">当月</van-radio>
        <van-radio name="3">当日</van-radio>
        <van-radio name="4">当年</van-radio>
      </van-radio-group>
      </div>
      <div style="flex: 2">
        <van-cell title="选择日期" :value="date" @click="show = true" style="display:block" />
        <van-calendar
          v-model="show"
          type="range"
          @confirm="onConfirm"
          :show-confirm="false"
          style="display:block"
        />
      </div>
    </div>
    <div class="mainContent1" style="display: flex">
      <!-- <van-cell-group inset>
        <van-cell title="总计划" value="1000" />
      </van-cell-group> -->
      <div style="flex: 2" class="groove">
        <div class="number">总计划</div>
        <div class="name">122</div>
      </div>

      <div style="flex: 5">
        <div style="float: left">
          <div>已完成</div>
          <div ref="finishChart" style="width: 120px; height: 120px"></div>
        </div>
        <div style="float: left">
          <div>未完成</div>
          <div ref="unfinishChart" style="width: 120px; height: 120px"></div>
        </div>
      </div>
    </div>
    <div class="mainContent2">
      <div ref="barChart" style="width: 200px; height: 150px"></div>
    </div>
    <div class="bottomContent">
      <div>员工计划完成情况</div>
      <el-table :data="tableData" stripe style="width: 100%">
        <el-table-column prop="xh" label="序号" width="50"> </el-table-column>
        <el-table-column prop="dept" label="部门" width="100">
        </el-table-column>
        <el-table-column prop="psn" label="工序责任人/分配人" width="70">
        </el-table-column>
        <el-table-column prop="finish" label="已完成" width="50">
        </el-table-column>
        <el-table-column prop="unfinish" label="未完成" width="50">
        </el-table-column>
        <el-table-column prop="finishR" label="完成率" width="50">
        </el-table-column>
      </el-table>
    </div>
    <!-- 数据加载提示gif -->
    <!-- <Loading :show="isShowLoading" /> -->
    <!-- 回到顶部按钮 -->
    <v-top />
  </div>
</template>

<script type="text/javascript">
import { getHomeData } from './../../serve/api/index.js'

// 引入Vuex
import { mapMutations, mapState } from 'vuex'

import { ADD_TO_CART } from './../../config/pubsub_type.js'
import $ from 'jquery';
// 引入页面组件
import Vue from 'vue';
import { Area } from 'vant';
import { Calendar } from 'vant';
import { Cell, CellGroup } from 'vant';
import * as echarts from 'echarts';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
Vue.use(Cell);
Vue.use(CellGroup);
Vue.use(Area);
Vue.use(Calendar);
Vue.use(ElementUI);
// import Header from './components/header/Header'
// import Sowing from './components/sowing/Sowing'
// import Tip from './components/tip/Tip'
// import Nav from './components/nav/Nav'
// import VipTip from './components/myVip/VipTip'
// import FlashBuy from './components/flash/FlashBuy'
// import SpecialZone from './components/special/SpecialZone'
// import TabbarGoodsItem from './components/tabbar/TabbarGoodsItem'
// import Loading from '../../components/loading/LoadingGif'

export default {
  computed: {
    ...mapState(['userInfo']),
  },
  created () {
    // 0.数据初始化
    this._initData();
  },
  mounted () {
    $(".van-picker__toolbar").remove();
    this.finishChart();
    this.unfinishChart();
    this.barChart();
  },
  data () {
    return {
      date: '',
      show: false,
      areaList: {
        province_list: {
          110000: '至今',
          120000: '当月',
          130000: '当日',
          130000: '当年',
        },
        // city_list: {
        //   110100: '北京市',
        //   120100: '天津市',
        //   130100: '济南市',
        //   130101: '烟台市',
        //   130102: '青岛市',
        // },
        // county_list: {
        //   110101: '东城区',
        //   110102: '西城区',
        //   120102: '西城区1',
        //   120102: '西城区1',
        //   130102: '开发区',
        //   130101: '高新区',
        //   130102: '芝罘区',
        //   // ....
        // },
      },
      radio: '1',
      finishNum: 95,
      unfinishNum: 5,
      totalhNum: 100,
      tableData: [{
        xh: '1',
        dept: '信息中心',
        psn: '王小虎',
        finish: 12,
        unfinish: 12,
        finishR: "12%",
      }, {
        xh: '1',
        dept: '上海市普陀区',
        psn: '王小虎',
        finish: 12,
        unfinish: 12,
        finishR: "12%",

      }, {
        xh: '1',
        dept: '上海市普陀区',
        psn: '王小虎',
        finish: 12,
        unfinish: 12,
        finishR: "12%",
      }, {
        xh: '1',
        dept: '上海市普陀区',
        psn: '王小虎',
        finish: 12,
        unfinish: 12,
        finishR: "12%",
      }]
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
    async _initData () {

    },
    timeFun () {
      console.log("快捷时间选择")
    },
    formatDate (date) {
      return `${date.getMonth() + 1}/${date.getDate()}`;
    },
    onConfirm (date) {
      const [start, end] = date;
      this.show = false;
      this.date = `${this.formatDate(start)} - ${this.formatDate(end)}`;
    },
    finishChart () {
      var that = this;
      let value = that.finishNum;
      let maxValue = that.totalhNum;
      let pipe = echarts.init(this.$refs.finishChart);
      let option = {
        title: {
          textStyle: {
            fontWeight: 'bold',
            fontFamily: 'Microsoft YaHei',
            fontSize: 24
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
              fontSize: 28
            }
          },
          //第二行文字
          //内容 + 位置
          {
            type: 'text',
            left: 'center',
            top: '55%',
            style: {
              //maxValue进度条最大值
              text: '共' + maxValue + '个',
              textAlign: 'center',
              fill: '#999',
              fontSize: 24
            }
          }
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
                  color: 'white'
                }
              }
            },
            //(maxValue进度条最大值 - value当前进度) + 颜色
            {
              value: maxValue - value,
              itemStyle: {
                normal: {
                  color: '#73A0FA'
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
    unfinishChart () {
      var that = this;
      let value = that.unfinishNum;
      let maxValue = that.totalhNum;
      let pipe = echarts.init(this.$refs.unfinishChart);
      let option = {
        title: {
          textStyle: {
            fontWeight: 'bold',
            fontFamily: 'Microsoft YaHei',
            fontSize: 24
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
              fontSize: 28
            }
          },
          //第二行文字
          //内容 + 位置
          {
            type: 'text',
            left: 'center',
            top: '55%',
            style: {
              //maxValue进度条最大值
              text: '共' + maxValue + '个',
              textAlign: 'center',
              fill: '#999',
              fontSize: 24
            }
          }
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
                  color: 'white'
                }
              }
            },
            //(maxValue进度条最大值 - value当前进度) + 颜色
            {
              value: maxValue - value,
              itemStyle: {
                normal: {
                  color: '#73A0FA'
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
    barChart () {
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
          data: ['有效载荷系统室', '有效载荷专业室']
        },
        yAxis: {
          type: 'value',
        },
        series: [
          {
            name: '已完成',
            type: 'bar',
            stack: 'total',
            label: {
              show: true
            },
            emphasis: {
              focus: 'series'
            },
            data: [320, 302]
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
            data: [120, 210]
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
  outline: 1px solid red;
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
// .van-picker__toolbar {
//   display: none !important;
// }

.groove {
  border-style: groove;
  height: 138px;
  width: 150px;
  background-color: red;
}
.topContent {
  height: 3.5rem;
}
.mainContent1 {
  height: 8rem;
}
.mainContent2 {
  height: 8rem;
}
.bottomContent {
  height: 8rem;
}
.el-table .el-table__cell {
  padding: 4px 0 !important;
}
.van-cell{
  padding: 5px 3px !important;
}
.van-cell__value{
  text-align: left;
}
</style>
