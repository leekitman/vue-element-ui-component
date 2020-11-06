<template>
  <div class="component" :style="{width: width, height: height}">
    <!-- 头部：显示当前年月，操作按钮 -->
    <div class="head-box">
      <div class="show-area">{{ year }} 年 {{ month + 1 }} 月</div>
      <div class="option-btns">
        <el-button icon="el-icon-arrow-left" @click="toLastMonth">上个月</el-button>
        <el-button @click="toToday">今天</el-button>
        <el-button @click="toNextMonth">下个月<i class="el-icon-arrow-right el-icon--right"></i></el-button>
      </div>
    </div>
    <!-- 日历主要显示区 -->
    <div class="main-box">
      <div class="week-title">
        <div class="title-items" v-for="weekIndex in weekIndexs" :key="'weekTitle'+weekIndex">
          <div class="title-item" v-for="weekTitle in weekTitles[weekIndex]" :key="weekTitle">
            {{ weekTitle }}
          </div>
        </div>
      </div>
      <div class="week-items">
        <div class="week-item" v-for="weekIndex in weekIndexs" :key="'weekItem'+weekIndex">
          <div class="date-item" v-for="date in getDatesInWeek(weekIndex)" :key="date.getTime()">
            <div
              :class="{
                'date-content': true,
                'out-month-color': !isCurrentMonth(date),
                'current-month-color': isCurrentMonth(date),
                'today-color': isToday(date)
            }">
              <slot name="dateContent" :date="date">
                {{ date.getDate() }}
              </slot>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
/**
 * 日历
 * @description 由于Element UI 的 Calendar组件在部署到服务器后，日期和星期就对应错误，解决不了，所以自己
 * @version 1.0.0
 *
 * @property {Number} firstDayOfWeek 起始周，可选项[0-6]，0是周日
 * @property {String} width 组件宽度
 * @property {String} height 组件高度
 * @property {Function} onCurrentMonthChange 当前日期变更时，主要是【上个月】【今天】【下个月】这三个按钮触发，或者未实现的选择日期触发
 */
export default {
  name: 'LjCalendar',
  props: {
    firstDayOfWeek: { type: Number, default: 1 },
    width: { type: String, default: '100%' },
    height: { type: String, default: null },
    onCurrentMonthChange: { type: Function, default: function(date) {} }
  },
  data: function() {
    return {
      today: new Date(),
      dateSelected: this.today,
      year: (new Date()).getFullYear(),
      month: (new Date()).getMonth(),
      weekTitles: ['日', '一', '二', '三', '四', '五', '六'],
      mondayDates: [],
      tuesdayDates: [],
      wednesdayDates: [],
      thursdayDates: [],
      fridayDates: [],
      saturdayDates: [],
      sundayDates: []
    }
  },
  computed: {
    weekIndexs() {
      var arr = new Array(7)
      var curr = this.firstDayOfWeek % 7
      for (var i = 0; i < 7; i++) {
        curr = (this.firstDayOfWeek + i) % 7
        arr[i] = curr
      }
      console.log('观察weekIndexs', arr)
      return arr
    }
  },
  created() {
    console.log('观察是否有插槽', this, this.$slots)
    this.generateDates(this.year, this.month)
  },
  methods: {
    generateDates(year, month) {
      // 清除旧数据
      this.mondayDates = []
      this.tuesdayDates = []
      this.wednesdayDates = []
      this.thursdayDates = []
      this.fridayDates = []
      this.saturdayDates = []
      this.sundayDates = []
      // 确认月首日星期几，是否需要往前填补
      var monthHeadDate = new Date(year, month, 1, 0, 0, 0, 0)
      var monthHeadDay = monthHeadDate.getDay()
      // console.log('生成日期列表：确认月首日星期：0-6：', monthHeadDate, monthHeadDay)
      // console.log('生成日期列表：确认起始周：0-6：', this.firstDayOfWeek)
      var startDate = monthHeadDate
      // 前推公式：n = (7 + h - f) % 7
      if (monthHeadDay !== this.firstDayOfWeek) {
        var hn = (7 + monthHeadDay - this.firstDayOfWeek) % 7
        startDate = new Date(monthHeadDate)
        startDate.setDate(monthHeadDate.getDate() - hn)
        // console.log('生成日期列表：确认前推几天：', hn, startDate)
      }
      // 确认月尾星期几，是否需要往后填补
      var monthTailDate = new Date(monthHeadDate)
      monthTailDate.setMonth(monthHeadDate.getMonth() + 1)
      monthTailDate.setDate(0)
      var monthTailDay = monthTailDate.getDay()
      // console.log('生日日期列表：确认月尾日星期：0-6：', monthTailDate, monthTailDay)
      var endDayOfWeek = (this.firstDayOfWeek + 6) % 7
      // console.log('生日日期列表：确认末位周：0-6：', endDayOfWeek)
      var endDate = monthTailDate
      // 后推公式：n = ( 7 + e - t ) % 7
      if (monthTailDay !== endDayOfWeek) {
        var en = (7 + endDayOfWeek - monthTailDay) % 7
        endDate = new Date(monthTailDate)
        endDate.setDate(monthTailDate.getDate() + en)
        // console.log('生成日期列表：确认后推几天：', en, endDate)
      }
      // 根据开始日和末位日生成日期列表
      var currentDate = startDate
      var addend = 0
      while (currentDate.getTime() !== endDate.getTime() && addend < 50) {
        //
        currentDate = new Date(startDate)
        currentDate.setDate(startDate.getDate() + addend)
        var currentWeek = currentDate.getDay()
        switch (currentWeek) {
          case 1 :
            this.mondayDates.push(currentDate)
            break
          case 2 :
            this.tuesdayDates.push(currentDate)
            break
          case 3 :
            this.wednesdayDates.push(currentDate)
            break
          case 4 :
            this.thursdayDates.push(currentDate)
            break
          case 5 :
            this.fridayDates.push(currentDate)
            break
          case 6 :
            this.saturdayDates.push(currentDate)
            break
          case 0 :
            this.sundayDates.push(currentDate)
            break
          default:
            console.error('错误的周期映射索引：', currentWeek)
        }
        addend = addend + 1
        // console.log('预测下一个循环条件：', currentDate, endDate, currentDate.getTime() !== endDate.getTime() && addend < 50)
      }
    },
    getDatesInWeek(weekIndex) {
      switch (weekIndex) {
        case 1 : return this.mondayDates
        case 2 : return this.tuesdayDates
        case 3 : return this.wednesdayDates
        case 4 : return this.thursdayDates
        case 5 : return this.fridayDates
        case 6 : return this.saturdayDates
        case 0 : return this.sundayDates
        default: return []
      }
    },
    isCurrentMonth(date) {
      return this.month === date.getMonth()
    },
    isToday(date) {
      var now = new Date()
      now.setHours(0)
      now.setMinutes(0)
      now.setSeconds(0)
      now.setMilliseconds(0)
      console.log('观察isToday:', now, date, now === date)
      return now.getTime() === date.getTime()
    },
    toLastMonth() {
      var currentMonthHead = new Date(this.year, this.month, 1, 0, 0, 0, 0)
      currentMonthHead.setMonth(currentMonthHead.getMonth() - 1)
      this.year = currentMonthHead.getFullYear()
      this.month = currentMonthHead.getMonth()
      this.generateDates(this.year, this.month)
      this.onCurrentMonthChange(currentMonthHead)
    },
    toToday() {
      var currentMonthHead = new Date()
      if (this.year === currentMonthHead.getFullYear() && this.month === currentMonthHead.getMonth()) {
        return
      }
      this.year = currentMonthHead.getFullYear()
      this.month = currentMonthHead.getMonth()
      this.generateDates(this.year, this.month)
      this.onCurrentMonthChange(currentMonthHead)
    },
    toNextMonth() {
      var currentMonthHead = new Date(this.year, this.month, 1, 0, 0, 0, 0)
      currentMonthHead.setMonth(currentMonthHead.getMonth() + 1)
      this.year = currentMonthHead.getFullYear()
      this.month = currentMonthHead.getMonth()
      this.generateDates(this.year, this.month)
      this.onCurrentMonthChange(currentMonthHead)
    }
  }
}
</script>
<style lang="scss">
.component{
}
.head-box{
  padding: 12px 20px;
  border-bottom: 1px solid #EBEEF5;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  .show-area{
    margin-right: 20px;;
  }
  .option-btns{
    margin-left: 20px;;
  }
}
.main-box{
  display: flex;
  flex-direction: column;
  width: 100%;
  padding: 12px 20px 35px;
  .week-title{
    display: flex;
    flex-direction: row;
    width: 100%;
    .title-items{
      width: 14.2857%;
      .title-item{
        height: 50px;
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
      }
    }
  }
  .week-items{
    display: flex;
    flex-direction: row;
    width: 100%;
    border-top:1px solid #EBEEF5;
    border-right:1px solid #EBEEF5;
    .week-item{
      width: 14.2857%;
      .date-item{
        border-left: 1px solid #EBEEF5;
        border-bottom: 1px solid #EBEEF5;
        padding: 8px;
        .date-content{
          height: 69px;
        }
      }
    }
  }
}
.out-month-color{
  color:#C0C4CC
}
.current-month-color{
  color: #000000
}
.today-color{
  color: #409EFF
}
</style>
