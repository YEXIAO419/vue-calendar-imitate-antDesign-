<template>
  <section class="wh_container">
    <div class="wh_content_all">
      <div class="wh_top_changge">
        <!--<div class="wh_content_li">{{dateTop}}</div>-->
        <div class="pre-next-month">
          <el-button size="mini" class="pre-month" @click="PreMonth(myDate,false)"><i class="el-icon-caret-left" /></el-button>
          <el-button size="mini" class="next-month" @click="NextMonth(myDate,false)" :disabled="myDate.getMonth() == (new Date()).getMonth()"><i class="el-icon-caret-right" /></el-button>
        </div>
      </div>
      <div class="wh_content week">
        <div class="wh_content_item" v-for="tag in textTop">
          <div class="wh_top_tag">
            {{tag}}
          </div>
        </div>
      </div>
      <div class="wh_content days">
        <div class="wh_content_item" v-for="(item,index) in list" @click="clickDay(item,index)" :class="[{ wh_isMark: item.isMark},{wh_chose_dayed:item.isChosedDayed}, {wh_sum_postion: item.sumPostion}]">
          <div class="wh_item_date" :class="[{wh_other_dayhide:item.otherMonth!=='nowMonth'},{wh_want_dayhide:item.dayHide},{wh_isToday:item.isToday},setClass(item)]">
            {{item.id}}日
          </div>
          <span class="wh_date-mark" v-if="!item.isMark">账单未生成</span>
          <span class="wh_download" v-if="item.isMark && item.isChosedDayed"><i class="el-icon-download" /></span>
          <div class="wh_sum" v-if="item.isMark && item.isChosedDayed">
            <el-row class="wh_sum-header">
              <el-col :span="8">余额账户</el-col>
              <el-col :span="8">收入余额账户</el-col>
              <el-col :span="8">余额变化</el-col>
            </el-row>
            <el-row class="wh_sum-item">
              <el-col :span="8" class="wh_sum-green">0.00 元</el-col>
              <el-col :span="8" class="wh_sum-red">0.00元</el-col>
              <el-col :span="8">0 笔</el-col>
            </el-row>
            <el-row class="wh_sum-item">
              <el-col :span="8">0 笔</el-col>
              <el-col :span="8">0 笔</el-col>
              <el-col :span="8">期末  000.</el-col>
            </el-row>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
<script>
import timeUtil from './calendar.js';

export default {
  data() {
    return {
      textTop: ['一', '二', '三', '四', '五', '六', '日'],
      myDate: [],
      list: [],
      historyChose: [],
      dateTop: ''
    };
  },
  props: {
    //标志特殊日子
    markDate: {
      type: Array,
      default: () => []
    },

    markDateMore: {
      type: Array,
      default: () => []
    },

    //选择固定月份
    choseDate: {type: Date},

    agoDayHide: { type: String, default: `0` },

    futureDayHide: { type: String, default: `2554387200` }
  },
  created() {
    this.myDate = new Date();
  },
  methods: {
    setClass(data) {
      let obj = {};
      obj[data.markClassName] = data.markClassName;
      return obj;
    },

    clickDay: function (item, index) {
      if (item.otherMonth === 'nowMonth' && !item.dayHide) {
        this.getList(this.myDate, item.date);
      }
      if (item.otherMonth !== 'nowMonth') {
        item.otherMonth === 'preMonth'
          ? this.PreMonth(item.date)
          : this.NextMonth(item.date);
      }
    },

    //选择固定月份
    ChoseMonth: function (date, isChosedDay = true) {
      date = timeUtil.dateFormat(date);
      this.myDate = new Date(date);
      if (isChosedDay) {
        this.getList(this.myDate, date, isChosedDay);
      } else {
        this.getList(this.myDate);
      }
    },

    //获取前一个月
    PreMonth: function (date, isChosedDay = true) {
      date = timeUtil.dateFormat(date);
      this.myDate = timeUtil.getOtherMonth(this.myDate, 'preMonth');
      this.$emit('changeMonth', timeUtil.dateFormat(this.myDate));
      if (isChosedDay) {
        this.getList(this.myDate, date, isChosedDay);
      } else {
        this.getList(this.myDate);
      }
    },

    //获取后一个月
    NextMonth: function (date, isChosedDay = true) {
      date = timeUtil.dateFormat(date);
      this.myDate = timeUtil.getOtherMonth(this.myDate, 'nextMonth');
      this.$emit('changeMonth', timeUtil.dateFormat(this.myDate));
      if (isChosedDay) {
        this.getList(this.myDate, date, isChosedDay);
      } else {
        this.getList(this.myDate);
      }
    },

    forMatArgs: function () {
      let markDate = this.markDate;
      for (let i = 0; i < markDate.length; i++) {
        markDate[i] = timeUtil.dateFormat(markDate[i]);
      }
      let markDateMore = this.markDateMore;
      for (let i = 0; i < markDateMore.length; i++) {
        markDateMore[i] = timeUtil.dateFormat(markDateMore[i]);
      }
      return [markDate, markDateMore];
    },
    
    getList: function (date, chooseDay, isChosedDay = true) {
      const [markDate, markDateMore] = this.forMatArgs();
      this.dateTop = `${date.getFullYear()}年${date.getMonth() + 1}月`;
      let arr = timeUtil.getMonthList(this.myDate);
      for (let i = 0; i < arr.length; i++) {
        let k = arr[i];
        k.chooseDay = false;
        const nowTime = k.date;
        const t = new Date(nowTime).getTime() / 1000;
        const d = new Date(nowTime).getDay();

        //账单汇总位置
        if(d == 0 || d == 6) {
          k.sumPostion = true;
        }else{
          k.sumPostion = false;
        }

        //标记选中某些天 设置class
        k.isMark = markDate.indexOf(nowTime) > -1;
        k.isChosedDayed = markDateMore.indexOf(nowTime) > -1; //自定义标志class ---已选中过的天数


        //无法选中某天
        k.dayHide = t < this.agoDayHide || t > this.futureDayHide;
        if (k.isToday) {
          this.$emit('isToday', nowTime);
        }
        let flag = !k.dayHide && k.otherMonth === 'nowMonth';
        if (chooseDay && chooseDay === nowTime && flag) {
          this.$emit('choseDay', nowTime);
          this.historyChose.push(nowTime);
          k.chooseDay = true;
          if(k.isMark && markDateMore.indexOf(nowTime) < 0){
            this.markDateMore.push(nowTime); //push选中的天数
            k.isChosedDayed = true;
          }
        } else if (
          this.historyChose[this.historyChose.length - 1] === nowTime && !chooseDay && flag
        ) {
          k.chooseDay = true;
        }
      }
      this.list = arr;
    }
  },
  mounted() {
    this.getList(this.myDate);
  },
  watch: {
    markDate(val, oldVal) {
      this.getList(this.myDate);
    },
    agoDayHide(val, oldVal) {
      this.agoDayHide = parseInt(val);
      this.getList(this.myDate);
    },
    futureDayHide(val, oldVal) {
      this.futureDayHide = parseInt(val);
      this.getList(this.myDate);
    },
    choseDate(val, oldVal){
      this.ChoseMonth(val);
    }
  }
};
</script>

<style scoped>
.wh_container {
  padding-top: 15px;
  margin: auto;
}

li {
  list-style-type: none;
}

.wh_top_changge {
  display: flex;
}

.wh_top_changge .wh_content_li {
  cursor: auto;
  flex: 2.5;
}

.wh_content_all {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC",
    "Helvetica Neue", STHeiti, "Microsoft Yahei", Tahoma, Simsun, sans-serif;
  width: 100%;
  overflow: hidden;
  padding-bottom: 8px;
}

.wh_content {
  display: flex;
  flex-wrap: wrap;
  width: 100%;
}

.wh_content:first-child .wh_content_item_tag,
.wh_content:first-child .wh_content_item {
  color: #ddd;
  font-size: 16px;
}

.wh_content_item,
wh_content_item_tag {
  font-size: 15px;
  width: 13.4%;
  text-align: center;
  color: #fff;
  position: relative;
}

.week .wh_content_item {
  background-color: #f6f6f6;
  color: #000;
  border-bottom: 1px solid #e6e6e6;
}

.days {
  border-left: 1px solid #e6e6e6;
}

.days .wh_content_item {
  padding: 30px 15px;
  border-collapse: collapse;
  border-bottom: 1px solid #e6e6e6;
  border-right: 1px solid #e6e6e6;
  background-color: #f6f6f6;
  position: relative;
}

.wh_top_tag {
  width: 40px;
  height: 40px;
  line-height: 40px;
  margin: auto;
  display: flex;
  justify-content: center;
  align-items: center;
}

.wh_item_date {
  width: 40px;
  height: 40px;
  line-height: 40px;
  margin: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #000;
}

.wh_content_item .wh_download {
  position: absolute;
  bottom: -1px;
  left: 8px;
  font-size: 20px;
  color: #000;
}

.wh_content_item:hover .wh_down {
  
}

.wh_content_item .wh_date-mark {
  position: absolute;
  font-size: 12px;
  bottom: 5px;
  color: #999;
  left: 10px;
  display: none;
}

.wh_content_item .wh_sum {
    position: absolute;
    font-size: 12px;
    color: #666;
    width: 360px;
    z-index: 1;
    background-color: #fff;
    border: solid 1px #ddd;
    box-shadow: 1px 1px 2px #ddd;
    border-radius: 5px;
    padding: 0 20px 5px 20px;
    text-align: left;
    bottom: -74px;
    display: none;
}

.wh_content_item.wh_sum_postion .wh_sum {
  right: 0;
}

.wh_content_item:hover .wh_sum {
  display: block;
}


.wh_sum .wh_sum-header {
  border-bottom: solid 1px #ddd;
  padding: 8px 0 2px 0;
  margin-bottom: 5px;
}

.wh_sum .wh_sum-green {
  color: #03be67;
}

.wh_sum .wh_sum-red {
  color: #e01515;
}

.wh_content_item:hover .wh_date-mark {
  display: block;
}

.pre-next-month {
  position: absolute;
  top: 12px;
  left: 370px;
}

.wh_content_item.wh_isMark {
  background: #fff;
}

.wh_content_item.wh_isMark:hover {
  background: #50E3C2;
  color: #fff;
  cursor: pointer;
}

.wh_content_item.wh_isMark:hover .wh_item_date, .wh_content_item.wh_isMark:hover .el-icon-download {
  color: #fff;
}

.wh_content_item .wh_other_dayhide {
  color: #bfbfbf;
}

.wh_content_item .wh_want_dayhide {
  color: #bfbfbf;
}

.wh_content_item .wh_isToday {
  background: yellow;
  border-radius: 100px;
}

.wh_content_item .wh_chose_day {
  background: green;
  border-radius: 100px;
}
</style>