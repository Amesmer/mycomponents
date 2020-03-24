<template>
  <div class="container">
    <div class="cusinfo">
      <div class="custitle">
        <img src="@/assets/pics/wodeyuyue备份@2x.png" alt />
        预约人信息
      </div>
      <div class="phonenum">
        <i>手机号</i>
        <span>136****9808</span>
      </div>
      <div class="checkcode">
        <i>验证码</i>
        <div>
          <input type="text" v-model="checkcode" placeholder="请输入验证码" />
          <span v-show="checkcodeshow" class="getcode" @click="getCode">获取验证码</span>
          <span v-show="!checkcodeshow" class="count">{{timecount}}s后重发</span>
        </div>
      </div>
    </div>
    <!-- 电话回访信息 -->
    <div class="cusinfo book">
      <div class="custitle">
        <img src="@/assets/pics/r@2x.png" alt />
        预约电话回访信息
      </div>
      <!-- 日期 -->
      <div class="date">
        <span class="leftdate">
          <i>*</i>预约日期
        </span>
        <span class="rightdate" @click="showdatePopup">
          <input type="text" placeholder="请选择" disabled :value="appointdate" />
          <img src="@/assets/pics/Chevron@2x.png" alt />
        </span>
        <!-- 时间 -->
      </div>
      <div class="date">
        <span class="leftdate">
          <i>*</i>预约时段
        </span>
        <span class="rightdate" @click="showtimePopup">
          <input type="text" placeholder="请选择" disabled :value="appointtime" />
          <img src="@/assets/pics/Chevron@2x.png" alt />
        </span>
      </div>
    </div>
    <!-- 提示 -->
    <p class="tips">
      <img src="@/assets/pics/编组 2@2x.png" alt />带*为必填项
    </p>

    <button v-show="!readyClick" class="booknow">立即预约</button>
    <button v-show="readyClick" class="booknow bookready" @click="sendbookmsg">立即预约</button>

    <!-- 日期弹出框 -->
    <van-popup v-model="dateshow">
      <van-calendar
        v-model="dateshow"
        :show-mark="false"
        @confirm="onConfirmdate"
        :poppable="false"
        :min-date="minDate"
        :max-date="maxDate"
      >
        <div slot="title">
          <div class="datetitle">
            <i @click="backdate">
              <img src="@/assets/pics/向右@2x(1).png" alt />
            </i>
            <span>{{currentmonth}}</span>
            <i @click="godate">
              <img src="@/assets/pics/向右@2x.png" alt />
            </i>
          </div>
        </div>
        <!-- <div slot="footer">
          <div class="datefooter">确定</div>
        </div>-->
      </van-calendar>
    </van-popup>

    <!-- 时间弹出框 -->
    <van-popup v-model="timeshow" position="bottom" :style="{ height: '35%' }">
      <van-picker
        show-toolbar
        title="请选择"
        default-index="3"
        :columns="pickeritems"
        @cancel="cancelchoosetime"
        @confirm="choosetime"
      />
    </van-popup>
  </div>
</template>

<script>
import { Popup } from "vant";
import { Calendar } from "vant";
import { DatetimePicker } from "vant";
import { Picker } from "vant";
export default {
  data() {
    return {
      // 验证码
      checkcode: "",
      checkcodeshow: true,
      timecount: "",
      timer: null,
      minDate: new Date(2020, 0, 1),
      maxDate: new Date(2020, 0, 31),
      monthindex: 0,
      currentmonth: "",
      // 预约时段
      appointtime: "",
      // 预约日期
      appointdate: "",
      timeshow: false,
      dateshow: false,
      pickeritems: [
        "8:30-9:30",
        "9:30-10:30",
        "10:30-11:30",
        "13:30-14:30",
        "14:30-15:30",
        "15:30-16:30",
        "16:30-17:30"
      ]
    };
  },
  computed: {
    readyClick() {
      // 非空判断
      if (!!this.checkcode && !!this.appointtime && !!this.appointdate) {
        return true;
      }
      return false;
    }
  },
  methods: {
    sendbookmsg() {
      this.$router.push({
        path: "/appointmentSuc",
        query: { appointtime: this.appointtime ,
            appointdate:this.appointdate
        }
      });
    },
    getCode() {
      const TIME_COUNT = 60;
      if (!this.timer) {
        this.timecount = TIME_COUNT;
        this.checkcodeshow = false;
        this.timer = setInterval(() => {
          if (this.timecount > 0 && this.timecount <= TIME_COUNT) {
            this.timecount--;
          } else {
            this.checkcodeshow = true;
            clearInterval(this.timer);
            this.timer = null;
          }
        }, 1000);
      }
    },
    // 后退月份
    backdate() {
      if (this.monthindex <= 0) {
        return;
      }
      var date = new Date();
      this.monthindex--;
      if (this.monthindex > 0) {
        this.minDate = new Date(
          date.getFullYear(),
          date.getMonth() + this.monthindex,
          1
        );
      } else {
        this.minDate = new Date(
          date.getFullYear(),
          date.getMonth() + this.monthindex,
          date.getDate()
        );
      }
      // 如果日历上显示的年大于当前年
      let calendaryear = this.currentmonth.split("-")[0];
      let calendarmonth = this.currentmonth.split("-")[1];
      console.log(
        "calendaryear:" + calendaryear + "calendarmonth" + calendarmonth
      );

      if (calendaryear > date.getFullYear()) {
        if (calendarmonth > 1) {
          this.currentmonth = calendaryear + "-" + (calendarmonth - 1);
        } else {
          this.currentmonth = calendaryear - 1 + "-" + 12;
        }
      } else {
        this.currentmonth = date.getFullYear() + "-" + (calendarmonth - 1);
      }
    },
    // 前进月份
    godate() {
      if (this.monthindex >= 12) {
        return;
      }
      let date = new Date();
      this.monthindex++;
      this.minDate = new Date(
        date.getFullYear(),
        date.getMonth() + this.monthindex,
        1
      );
      // console.log(date.getMonth()+this.monthindex+1);

      // 大于12月 +1年
      if (date.getMonth() + this.monthindex + 1 > 12) {
        this.currentmonth =
          date.getFullYear() +
          1 +
          "-" +
          ((date.getMonth() + this.monthindex + 1) % 12);
      } else {
        this.currentmonth =
          date.getFullYear() + "-" + (date.getMonth() + this.monthindex + 1);
      }
    },
    // 日期选择弹出
    showdatePopup() {
      this.dateshow = true;
      setTimeout(() => {
        this.dateshow = false;
        this.dateshow = true;
      }, 100);
    },
    // 时间选择弹出
    showtimePopup() {
      this.timeshow = true;
    },

    formatDate(date) {
      return `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;
    },
    // 确认时间按钮
    choosetime(value) {
      this.appointtime = value;
      this.timeshow = false;
    },
    // 取消选择时间
    cancelchoosetime() {
      this.timeshow = false;
    },
    // 日期选择确认
    onConfirmdate(date) {
      this.dateshow = false;
      this.appointdate = this.formatDate(date);
      console.log(this.appointdate);
    },
    // 过滤周末
    filterweekend() {
      let datelist = null;
      datelist = document.querySelectorAll(".van-calendar__day");
      let calendaryear = this.currentmonth.split("-")[0];
      let calendarmonth = this.currentmonth.split("-")[1];
      // console.log(datelist);

      datelist.forEach(item => {
        let day = item.innerText;
        let weekdaytest = new Date(calendaryear, calendarmonth - 1, day);
        // console.log(weekdaytest);

        let isweek = weekdaytest.getDay();

        if (isweek == 0 || isweek == 6 || weekdaytest < new Date()) {
          item.style.setProperty("pointer-events", "none", "important");
          item.style.setProperty("color", "#c8c9cc");
          // item.style.setProperty('cursor','defaule')
        } else {
          item.style.setProperty("pointer-events", "auto", "important");
          item.style.setProperty("color", "#393c3c");
        }
      });
    }
  },
  mounted() {},
  updated() {
    this.filterweekend();
  },
  created() {
    //  minDate: new Date(2010, 0, 1),
    var date = new Date();
    this.currentmonth = date.getFullYear() + "-" + (date.getMonth() + 1);

    this.minDate = new Date(
      date.getFullYear(),
      date.getMonth(),
      date.getDate()
    );
    this.maxDate = new Date(
      date.getFullYear(),
      date.getMonth() + 12,
      date.getDate()
    );
  },
  //创建前设置
  beforeCreate() {
    document
      .querySelector("body")
      .setAttribute("style", "background-color:#f5f5f5;");
  },
  //销毁前清除
  beforeDestroy() {
    document.querySelector("body").removeAttribute("style");
  }
};
</script>

<style scoped lang="scss">
.container {
  background-color: #f5f5f5;
  .cusinfo {
    padding: 0 0.28rem;
    background-color: #fff;
    .custitle img {
      width: 0.36rem;
      height: 0.34rem;
      margin-right: 0.2rem;
    }
    .custitle {
      padding: 0.23rem 0;
      color: #00ae4d;
      font-size: 0.28rem;
      font-family: PingFangSC-Medium, PingFang SC;
      font-weight: 500;
      color: rgba(0, 174, 77, 1);
      line-height: 0.28rem;
      border-bottom: 1px solid #cdd5d2;
    }

    .phonenum,
    .checkcode {
      margin: 0.32rem 0;
      color: #848988;
      font-size: 0.0028rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .checkcode {
      padding-bottom: 0.19rem;
    }
    .phonenum span {
      height: 0.28rem;
      font-size: 0.28rem;
      font-family: PingFangSC-Regular, PingFang SC;
      font-weight: 400;
      color: rgba(57, 60, 60, 1);
      line-height: 0.28rem;
    }
    .checkcode input {
      height: 0.54rem;
      line-height: 0.54rem;
      width: 1.68rem;
      background-color: #fff;
    }
    .checkcode input::placeholder {
      color: rgba(196, 199, 197, 1);
    }
    .checkcode span {
      color: #00ae4d;
      display: inline-block;
      text-align: center;
      line-height: 0.52rem;
      width: 1.58rem;
      height: 0.54rem;
      border-radius: 0.08rem;
      border: 1px solid rgba(2, 174, 77, 1);
    }
    .checkcode .count {
      font-size: 0.24rem;
      color: rgba(164, 165, 165, 1);
      border: 1px solid rgba(164, 165, 165, 1);
    }
  }
  .book {
    margin-top: 0.2rem;

    .date {
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 0.28rem;
      .leftdate,
      .rightdate {
        height: 0.84rem;
        font-size: 0.28rem;
        font-family: PingFangSC-Regular, PingFang SC;
        font-weight: 400;
        color: #848988;
        line-height: 0.84rem;
      }
      .rightdate {
        color: rgba(196, 199, 197, 1);
      }
      .rightdate input {
        width: 1.57rem;
        background-color: #fff;
        text-align: right;
        color: #393c3c;
      }
      .rightdate input::placeholder {
        color: rgba(196, 199, 197, 1);
      }
      .leftdate i {
        color: #f8631a;
      }
      .rightdate img {
        width: 0.14rem;
        height: 0.26rem;
      }
    }
  }
  .tips {
    font-size: 0.24rem;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: rgba(132, 137, 136, 1);
    height: 0.36rem;
    line-height: 0.36rem;
    padding-top: 0.16rem;
    img {
      margin: 0 0.06rem 0 0.28rem;
      width: 0.24rem;
      height: 0.24rem;
    }
  }
  .booknow {
    margin: 0.84rem 0.28rem 0;
    width: 6.94rem;
    height: 0.88rem;
    font-size: 0.34rem;
    text-align: center;
    color: #fff;
    background: rgba(196, 240, 215, 1);
    border-radius: 0.08rem;
  }
  .bookready {
    background: rgba(0, 174, 77, 1);
    color: #ffffff;
  }
}

/deep/ .van-picker .van-picker-column__item {
  padding: 0 5px;
  font-size: 0.28rem !important;
  color: #393c3c !important;
}
/deep/ .van-picker .van-picker-column__item--selected {
  color: #00ae4d !important;
}

/deep/ .van-picker .van-picker__cancel {
  color: #393c3c;
  font-size: 0.28rem;
  font-family: PingFangSC-Regular, PingFang SC;
}

/deep/ .van-picker .van-picker__confirm {
  font-size: 0.28rem;
  font-family: PingFangSC-Regular, PingFang SC;
  color: #00ae4d;
}

/deep/ .van-picker .van-picker__title {
  font-size: 0.32rem;
  font-family: PingFangSC-Medium, PingFang SC;
  font-weight: 500;
}

/deep/ .van-calendar {
  width: 6.6rem;
  height: 6.92rem;
  border-radius: 0.08rem !important;
  overflow: hidden;
}
/deep/ .van-calendar .van-calendar__month-title {
  display: none;
}
/deep/ .van-calendar .van-calendar__footer {
  padding: 0;
}
/deep/ .van-popup--center {
  border-radius: 0.08rem !important;
}
/deep/ .van-calendar .van-calendar__day {
  height: 0.73rem;
  line-height: 0.73rem;
  font-size: 0.3rem;
}
/deep/ .van-calendar .van-calendar__days {
  height: 4.4rem;
}

/deep/ .van-calendar .van-calendar__selected-day {
  width: 0.5rem;
  height: 0.5rem;
  border-radius: 50%;
  background: linear-gradient(
    225deg,
    rgba(56, 239, 125, 1) 0%,
    rgba(17, 153, 142, 1) 100%
  );
}
/deep/ .van-calendar .van-calendar__body {
  background-color: #fff;
  overflow: hidden;
}
/deep/ .van-calendar .van-calendar__confirm {
  width: 6.6rem;
  height: 0.88rem;
  line-height: 0.88rem;
  text-align: center;
  background: rgba(0, 174, 77, 1);
  color: rgba(255, 255, 255, 1);
  font-size: 0.32rem;
  border: none;
  border-radius: 0;
  margin-bottom: 0;
}

// /deep/ .van-calendar__days div:nth-child(7n + 0) {
//   color: #c8c9cc;
//   cursor: default;
//   pointer-events: none;
// }
// /deep/ .van-calendar__days div:nth-child(7n + 8) {
//   color: #c8c9cc;
//   cursor: default;
//   pointer-events: none;
// }
.datetitle {
  height: 0.86rem;
  display: flex;
  justify-content: space-between;
  background: linear-gradient(
    225deg,
    rgba(56, 239, 125, 1) 0%,
    rgba(17, 153, 142, 1) 100%
  );
}
.datefooter {
  width: 6.6rem;
  height: 0.88rem;
  line-height: 0.88rem;
  text-align: center;
  background: rgba(0, 174, 77, 1);
  color: rgba(255, 255, 255, 1);
  font-size: 0.32rem;
}

.datetitle img {
  width: 0.14rem;
  height: 0.28rem;
  padding: 0 0.78rem;
}

.datetitle span {
  font-size: 0.32rem;
  font-family: PingFangSC-Medium, PingFang SC;
  font-weight: 500;
  color: rgba(255, 255, 255, 1);
}
</style>
