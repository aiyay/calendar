<template>
  <div class="date-time">
    <div class="nav-tips">
      <p class="">因铁路局列车运行调整，火车票预售期调整为30天，建议您提前预约抢票，开售自动抢票。</p>
      <div class="date-week-group">
        <div class="date-week-item" :class="{'red': index === 0||index === 6}" v-for="(item, index) in week" :key="index">{{item}}</div>
      </div>
    </div>
    <div class="block"></div>
    <div class="date-content">
      <div class="date-day-group" v-for="index in dateObj.monthRange" :key="index">
        <div class="date-day-title">
          {{ dateObj.dateTitle[index - 1] }}
        </div>
        <div class="date-day-content">
          <div @click="toSelectDate(index - 1, idx)" class="date-day-item" :class="[item.style, {active: item.isActive} ]" v-for="(item, idx) in dateObj.dayinfo[index - 1].dayList" :key="idx">
            <p class="red">{{item.tips}}</p>
            <p>{{item.val}}</p>
            <p class="red">{{item.label}}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import calendar from '@/utils/calendar.js'

export default {
  name: 'DateTime',
  data () {
    return {
      week: ['日', '一', '二', '三', '四', '五', '六'],
      month: [],
      dateObj: {
        monthRange: 0,
        dateTitle: [],
        dayinfo: []
      },
      festival: {
        lunar: {
          '1-1': '春节',
          '1-15': '元宵节',
          '2-2': '龙头节',
          '5-5': '端午节',
          '7-7': '七夕节',
          '7-15': '中元节',
          '8-15': '中秋节',
          '9-9': '重阳节',
          '10-1': '寒衣节',
          '10-15': '下元节',
          '12-8': '腊八节',
          '12-23': '祭灶节'
        },
        gregorian: {
          '1-1': '元旦',
          '2-14': '情人节',
          '3-8': '妇女节',
          '3-12': '植树节',
          '4-5': '清明节',
          '5-1': '劳动节',
          '5-4': '青年节',
          '6-1': '儿童节',
          '7-1': '建党节',
          '8-1': '建军节',
          '9-10': '教师节',
          '10-1': '国庆节',
          '12-24': '平安夜',
          '12-25': '圣诞节'
        }
      }
    }
  },
  methods: {
    // 标记日期
    markDate (day) {
      for (let i = 0; i < this.dateObj.dayinfo.length; i++) {
        for (let j = 0; j < this.dateObj.dayinfo[i].dayList.length; j++) {
          if (this.dateObj.dayinfo[i].dayList[j].day === day) {
            this.dateObj.dayinfo[i].dayList[j].isActive = true
            break
          }
        }
      }
    },
    // 选择日期
    toSelectDate (index, idx) {
      if (this.dateObj.dayinfo[index].dayList[idx].style === 'grey') return
      for (let i = 0; i < this.dateObj.dayinfo.length; i++) {
        for (let j = 0; j < this.dateObj.dayinfo[i].dayList.length; j++) {
          this.dateObj.dayinfo[i].dayList[j].isActive = false
        }
      }
      this.dateObj.dayinfo[index].dayList[idx].isActive = true
      this.$forceUpdate()
    },
    // 获取所选月份的最后一天。
    getMonthLast (year, month) {
      let lastdate = new Date(year, month, 0)
      return lastdate
    },
    // 获取所选月份的第一天。
    getMonthFirst (year, month) {
      let startdate = new Date(year, month - 1, 1)
      return startdate
    },
    // 获取所选月份的总天数。
    getMonthCountDays (year, month) {
      let date = new Date(year, month, 0)
      var days = date.getDate()
      return days
    },
    /* 构建售票日历
     * @params day: 预售时间段
     * @params robday: 抢票时间段
     */
    getDateTime (day, robday) {
      let myDate = new Date()
      let oldMonth = myDate.getMonth() + 1
      let oldYear = myDate.getFullYear()
      let oldDate = myDate.getDate()
      myDate.setDate(myDate.getDate() + day - 1 + robday)
      let newMonth = myDate.getMonth() + 1
      // 预售时间段跨度几个月
      this.dateObj.monthRange = (newMonth - oldMonth < 0 ? newMonth - oldMonth + 12 : newMonth - oldMonth) + 1
      let _preDays = 0 // 第一个月的跨度剩余下的数量
      let _preMount = 0 // 其后月的跨度剩余下的数量
      for (let i = 0; i < this.dateObj.monthRange; i++) {
        let _month = oldMonth + i
        let _year = oldYear
        // 跨年的月份判断
        if (_month > 12) {
          _year = _year + ~~(_month / 12)
          _month = _month % 12
          let countDays = this.getMonthCountDays(_year, _month)
          let dayList = []
          let firstDay = this.getMonthFirst(_year, _month).getDay()
          let lastDay = this.getMonthLast(_year, _month).getDay()
          // 每个月天数
          for (let j = 0; j < countDays; j++) {
            let _date = new Date(_year, _month - 1, j + 1).getDay()
            let _isWeek = false
            if (_date === 0 || _date === 6) {
              _isWeek = true
            } else {
              _isWeek = false
            }
            // 计算农历新历节日
            let lunarInfo = calendar.solar2lunar(_year, _month, j + 1)
            let lunarValue = ''
            if (this.festival.lunar[lunarInfo.lMonth + '-' + lunarInfo.lDay] !== undefined) {
              lunarValue = this.festival.lunar[lunarInfo.lMonth + '-' + lunarInfo.lDay]
            } else if (this.festival.gregorian[_month + '-' + (j + 1)] !== undefined) {
              lunarValue = this.festival.gregorian[_month + '-' + (j + 1)]
            }
            // 今天日期
            if (i === 0 && j + 1 === oldDate) {
              dayList[j] = {
                val: j + 1,
                label: '今',
                tips: lunarValue,
                style: 'red',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
              continue
            }
            // 预售跨期
            if ((i === 0 && day + oldDate > j + 1 && j + 1 > oldDate) || (i > 0 && _preMount > j)) {
              dayList[j] = {
                val: j + 1,
                label: '',
                tips: lunarValue,
                style: _isWeek ? 'red' : '',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
            } else {
              // 抢票跨期
              if ((i === 0 && day + oldDate <= j + 1 && j + 1 < day + oldDate + robday) || (i > 0 && _preMount + robday > j)) {
                dayList[j] = {
                  val: j + 1,
                  label: '抢',
                  tips: lunarValue,
                  style: _isWeek ? 'red' : '',
                  isActive: false,
                  day: _year + '-' + _month + '-' + (j + 1)
                }
                continue
              }
              // 不属于售票跨期
              dayList[j] = {
                val: j + 1,
                label: '',
                tips: lunarValue,
                style: 'grey',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
            }
            // 明天
            if (i === 0 && j === oldDate) {
              dayList[j] = {
                val: j + 1,
                label: '明',
                tips: lunarValue,
                style: 'red',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
              continue
            }
          }
          // 前面日历填充
          for (let j = 0; j < firstDay; j++) {
            dayList.unshift({
              val: '',
              label: '',
              tips: '',
              style: '',
              isActive: false,
              day: ''
            })
          }
          // 后面日历填充
          for (let j = 0; j < lastDay; j++) {
            dayList.push({
              val: '',
              label: '',
              tips: '',
              style: '',
              isActive: false,
              day: ''
            })
          }
          this.dateObj.dayinfo[i] = {
            dayList: dayList
          }
          // 计算剩余售票期
          if (i === 0) {
            _preDays = countDays - oldDate
            _preMount = day - _preDays - 1
          } else {
            _preMount = _preMount - countDays
          }
          if (_month < 10) {
            _month = '0' + _month
          }
          this.dateObj.dateTitle[i] = `${_year}年${_month}月`
        } else {
          // 未跨年的月
          let countDays = this.getMonthCountDays(_year, _month)
          let dayList = []
          let firstDay = this.getMonthFirst(_year, _month).getDay()
          let lastDay = this.getMonthLast(_year, _month).getDay()
          for (let j = 0; j < countDays; j++) {
            let _date = new Date(_year, _month - 1, j + 1).getDay()
            let _isWeek = false
            if (_date === 0 || _date === 6) {
              _isWeek = true
            } else {
              _isWeek = false
            }
            let lunarInfo = calendar.solar2lunar(_year, _month, j + 1)
            let lunarValue = ''
            if (this.festival.lunar[lunarInfo.lMonth + '-' + lunarInfo.lDay] !== undefined) {
              lunarValue = this.festival.lunar[lunarInfo.lMonth + '-' + lunarInfo.lDay]
            } else if (this.festival.gregorian[_month + '-' + (j + 1)] !== undefined) {
              lunarValue = this.festival.gregorian[_month + '-' + (j + 1)]
            }
            if (i === 0 && j + 1 === oldDate) {
              dayList[j] = {
                val: j + 1,
                label: '今',
                tips: lunarValue,
                style: 'red',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
              continue
            }
            if ((i === 0 && day + oldDate > j + 1 && j + 1 > oldDate) || (i > 0 && _preMount > j)) {
              dayList[j] = {
                val: j + 1,
                label: '',
                tips: lunarValue,
                style: _isWeek ? 'red' : '',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
            } else {
              if ((i === 0 && day + oldDate <= j + 1 && j + 1 < day + oldDate + robday) || (i > 0 && _preMount + robday > j)) {
                dayList[j] = {
                  val: j + 1,
                  label: '抢',
                  tips: lunarValue,
                  style: _isWeek ? 'red' : '',
                  isActive: false,
                  day: _year + '-' + _month + '-' + (j + 1)
                }
                continue
              }
              dayList[j] = {
                val: j + 1,
                label: '',
                tips: lunarValue,
                style: 'grey',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
            }
            if (i === 0 && j === oldDate) {
              dayList[j] = {
                val: j + 1,
                label: '明',
                tips: lunarValue,
                style: 'red',
                isActive: false,
                day: _year + '-' + _month + '-' + (j + 1)
              }
              continue
            }
          }
          for (let j = 0; j < firstDay; j++) {
            dayList.unshift({
              val: '',
              label: '',
              tips: '',
              style: '',
              isActive: false,
              day: ''
            })
          }
          for (let j = 0; j < lastDay; j++) {
            dayList.push({
              val: '',
              label: '',
              tips: '',
              style: '',
              isActive: false,
              day: ''
            })
          }
          this.dateObj.dayinfo[i] = {
            dayList: dayList
          }
          if (i === 0) {
            _preDays = countDays - oldDate
            _preMount = day - _preDays - 1
          } else {
            _preMount = _preMount - countDays
          }
          if (_month < 10) {
            _month = '0' + _month
          }
          this.dateObj.dateTitle[i] = `${_year}年${_month}月`
        }
      }
      console.log(this.dateObj)
    }
  },
  mounted () {
    this.getDateTime(30, 7)
    this.markDate('2018-12-12')
  }
}
</script>
<style lang="less">
@import './DateTime';
</style>


