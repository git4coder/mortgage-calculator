<template>
  <div class="container">
    <div class="charts">
      <dl class="chart">
        <dt>综合</dt>
        <dd>
          <ve-line :data="paymentRatio_loanInterest"></ve-line>
        </dd>
      </dl>
      <!-- <dl class="chart">
        <dt>各首付比例的利息差</dt>
        <dd>
          <ve-line :data="paymentRatio_loanInterest_difference"></ve-line>
        </dd>
      </dl> -->
      <dl class="chart">
        <dt>各首付比例的月供</dt>
        <dd>
          <ve-line :data="paymentRatio_loanPreMonth"></ve-line>
        </dd>
      </dl>
      <dl class="chart">
        <dt>预计房价走势</dt>
        <dd>
          <ve-line :data="housingPrice_year"></ve-line>
        </dd>
      </dl>
      <dl class="chart">
        <dt>赚钱速度和预计房价涨速</dt>
        <dd>
          <ve-line :data="housingPriceDifference_income"></ve-line>
        </dd>
      </dl>
      <dl class="chart">
        <dt>贷款年限与利息(首付{{paymentRatio}}%)</dt>
        <dd>
          <ve-line :data="loanTerm_loanInterest"></ve-line>
        </dd>
      </dl>
    </div>
    <div class="control" v-if="true">
      <group title="设置(等额本息)">
        <x-number title="面积(平)" :step="0.1" :min="0" v-model="area" width="80px" :fillable="true"></x-number>
        <x-number title="单价(元)" :step="1000" :min="0" v-model="price" width="80px" :fillable="true"></x-number>
        <x-number title="首付(%)" :step="1" :min="0" v-model="paymentRatio" width="80px" :fillable="true"></x-number>
        <cell title="总价" :value="housingPrice"></cell>
        <cell title="贷款" :value="loan"></cell>
        <cell title="首付" :value="downpayment"></cell>
        <cell title="月供" :value="loanPreMonth"></cell>
        <cell title="利息" :value="loanInterest"></cell>
        <x-number title="贷款利率(%)" :step="0.1" :min="0" v-model="loanRate" width="80px" :fillable="true"></x-number>
        <x-number title="贷款时长(年)" :step="1" :min="0" v-model="loanTerm" width="80px" :fillable="true"></x-number>
      </group>
      <group title="预测">
        <x-number title="涨幅(元/m²/年)" :step="100" :min="0" v-model="priceGain" width="80px" :fillable="true"></x-number>
        <x-number title="年收入(元)" :step="1000" :min="0" v-model="income" width="80px" :fillable="true"></x-number>
        <x-number title="时间跨度(年)" :step="1" :min="0" v-model="timeSpan" width="80px" :fillable="true"></x-number>
      </group>
    </div>
  </div>
</template>

<script>
import { Group, Cell, XNumber } from 'vux';
export default {
  components: {
    Group,
    Cell,
    XNumber
  },
  data() {
    return {
      area: 91.7, // 面积m2
      price: 11400, // 单价m2/元
      priceGain: 500, // 涨幅m2/元
      income: 70000, // 年收入
      timeSpan: 10, // 时间跨度（前后各几年）
      loanTerm: 30, // 贷款年限时长
      loanRate: 6.3, // 贷款利率%
      paymentRatio: 30 // 首付%
    };
  },
  computed: {
    housingPrice() {
      var that = this;
      var totalPrice = that.area * that.price;
      return totalPrice;
    },
    // 贷款
    loan() {
      var that = this;
      var housingPrice = that.housingPrice;
      var paymentRatio = parseInt(that.paymentRatio);
      return housingPrice - housingPrice * paymentRatio / 100;
    },
    // 首付
    downpayment() {
      var that = this;
      var value = that.housingPrice - that.loan;
      value = value.toFixed(2);
      value = parseFloat(value);
      return value;
    },
    // 每月应还贷款
    loanPreMonth() {
      var that = this;
      var loan = that.loan; // 贷款
      var loanTerm = parseInt(that.loanTerm); // 贷款年限时长
      var monthRate = parseFloat(that.loanRate) / 100 / 12; // 贷款利率%
      var pow = Math.pow(1 + monthRate, loanTerm * 12);
      var value = monthRate * pow / (pow - 1) * loan; // https://baike.baidu.com/item/等额本息/3227456
      return value.toFixed(2);
    },
    // 贷款利息
    loanInterest() {
      var that = this;
      var loan = parseInt(that.loan);
      var loanTerm = parseInt(that.loanTerm) * 12;
      var loanRate = parseFloat(that.loanRate) / 100 / 12;
      var loanPreMonth = parseFloat(that.loanPreMonth);
      var totalInterest = 0;
      for (let i = 1; i <= loanTerm; i++) {
        var interest = loan * loanRate; // 本月利息
        var payment = loanPreMonth - interest; // 本月本金
        loan -= payment; // 剩余本金
        totalInterest += interest;
        // console.log(i, loanPreMonth, payment, interest, loan);
      }
      return totalInterest.toFixed(2);
    },
    housingPriceDifference_income() {
      var that = this;
      var data = {
        columns: ['时间', '累计收入', '累计上涨', '累计利息', '上涨+利息', '首付'],
        rows: []
      };
      var year = new Date().getFullYear();
      var timeSpan = that.timeSpan;
      var price = that.price;
      var housingPrice = that.getHousingPrice(price);
      var loan = that.getLoan(that.paymentRatio, housingPrice);
      var loanPreMonth = that.getLoanPreMonth(loan, that.loanTerm, that.loanRate);
      var loanInterest = that.getLoanInterest(loan, that.loanTerm, that.loanRate, loanPreMonth);
      var loanInterestNew = loanInterest;
      for (let i = year; i < year + timeSpan; i++) {
        housingPrice = that.getHousingPrice(price);
        loan = that.getLoan(that.paymentRatio, housingPrice);
        loanPreMonth = that.getLoanPreMonth(loan, that.loanTerm, that.loanRate);
        loanInterestNew = that.getLoanInterest(loan, that.loanTerm, that.loanRate, loanPreMonth);
        data.rows.push({
          '时间': i + '',
          '累计收入': that.income * (i - year),
          '累计上涨': price * that.area - that.price * that.area,
          '累计利息': loanInterestNew - loanInterest,
          '上涨+利息': price * that.area - that.price * that.area + loanInterestNew - loanInterest,
          '首付': price * that.area - loan
        });
        price += that.priceGain;
      }
      return data;
    },
    housingPrice_year() {
      var that = this;
      var data = {
        columns: ['年', '总价', '利息'],
        rows: []
      };
      var year = new Date().getFullYear();
      var timeSpan = that.timeSpan;
      var price = that.price;
      for (let i = year; i < year + timeSpan; i++) {
        var housingPrice = that.getHousingPrice(price);
        var loan = that.getLoan(that.paymentRatio, housingPrice);
        var loanPreMonth = that.getLoanPreMonth(loan, that.loanTerm, that.loanRate);
        var loanInterest = that.getLoanInterest(loan, that.loanTerm, that.loanRate, loanPreMonth);
        console.log('loanInterest:', loanInterest);
        data.rows.push({
          '年': i + '',
          '总价': housingPrice,
          // '贷款': loan,
          '利息': loanInterest
        });
        price += that.priceGain;
      }
      return data;
    },
    // 首付-月供
    paymentRatio_loanPreMonth() {
      var that = this;
      var data = {
        columns: ['首付', '月供'],
        rows: []
      };
      for (let i = 30; i <= 70; i += 5) {
        var loan = that.getLoan(i, that.housingPrice); // 首付比例 i% 后的贷款金额
        var loanPreMonth = that.getLoanPreMonth(loan, that.loanTerm, that.loanRate);
        data.rows.push({
          '首付': i + '%',
          '月供': loanPreMonth
        });
      }
      // console.log(data);
      return data;
    },
    // 首付利息差
    paymentRatio_loanInterest_difference() {
      var that = this;
      var data = {
        columns: ['首付', '利息差'],
        rows: []
      };
      var loanInterestOld = 0;
      var loanInterestNew = 0;
      for (let i = 30; i <= 50; i++) {
        var loan = that.getLoan(i, that.housingPrice); // 首付比例 i% 后的贷款金额
        var loanPreMonth = that.getLoanPreMonth(loan, that.loanTerm, that.loanRate);
        loanInterestNew = that.getLoanInterest(loan, that.loanTerm, that.loanRate, loanPreMonth);
        data.rows.push({
          '首付': i + '%',
          '利息差': loanInterestOld ? loanInterestOld - loanInterestNew : loanInterestOld
        });
        loanInterestOld = loanInterestNew;
      }
      // console.log(data);
      return data;
    },
    // 贷款时间与利息
    loanTerm_loanInterest() {
      var that = this;
      var data = {
        columns: ['年限', '利息'],
        rows: []
      };
      for (let i = 1; i <= 30; i++) {
        var loan = that.getLoan(that.paymentRatio, that.housingPrice); // 首付比例
        var loanPreMonth = that.getLoanPreMonth(loan, i, that.loanRate);
        var loanInterest = that.getLoanInterest(loan, i, that.loanRate, loanPreMonth);
        data.rows.push({
          '年限': i + '年',
          '利息': loanInterest
        });
      }
      // console.log(data);
      return data;
    },
    // 综合
    paymentRatio_loanInterest() {
      var that = this;
      var data = {
        columns: ['比例', '首付', '利息', '贷款', '贷款+利息', '房价'],
        rows: []
      };
      for (let i = 0; i <= 100; i += 5) {
        var loan = that.getLoan(i, that.housingPrice); // 首付比例 i% 后的贷款金额
        var loanPreMonth = that.getLoanPreMonth(loan, that.loanTerm, that.loanRate);
        var loanInterest = that.getLoanInterest(loan, that.loanTerm, that.loanRate, loanPreMonth);
        var totalCost = loan + loanInterest;
        // console.log(i, loanPreMonth, '>>>>', loan, that.loanTerm);
        data.rows.push({
          '比例': i + '%',
          '首付': that.housingPrice - loan,
          '利息': loanInterest,
          '贷款': loan,
          '贷款+利息': totalCost,
          '房价': that.housingPrice
        });
      }
      // console.log('paymentRatio_loanInterest:', data.rows);
      return data;
    }
  },
  methods: {
    getHousingPrice(price) {
      var that = this;
      var totalPrice = that.area * price;
      return totalPrice;
    },
    // 贷款
    getLoan(paymentRatio, housingPrice) { // paymentRatio 首付比例
      paymentRatio = parseInt(paymentRatio);
      return housingPrice - housingPrice * paymentRatio / 100;
    },
    // 每月应还贷款
    getLoanPreMonth(loan, loanTerm, loanRate) {
      // var loan = 731766, loanTerm = 30, loanRate = 6.3;
      var monthRate = parseFloat(loanRate) / 100 / 12; // 年利率%转换为月利率
      var pow = Math.pow(1 + monthRate, loanTerm * 12);
      var value = monthRate * pow / (pow - 1) * loan; // https://baike.baidu.com/item/等额本息/3227456
      // console.log(loan, loanTerm, loanRate, value.toFixed(2));
      return value.toFixed(2);
    },
    // 总利息
    getLoanInterest(loan, loanTerm, loanRate, loanPreMonth) {
      var totalInterest = 0;
      loan = parseInt(loan);
      loanTerm = parseInt(loanTerm) * 12;
      loanRate = parseFloat(loanRate) / 100 / 12;
      loanPreMonth = parseFloat(loanPreMonth);
      for (let i = 1; i <= loanTerm; i++) {
        var interest = loan * loanRate; // 本月利息
        var payment = loanPreMonth - interest; // 本月本金
        loan -= payment; // 剩余本金
        totalInterest += interest;
        // console.log(i, loanPreMonth, payment, interest, loan);
      }
      return parseFloat(totalInterest.toFixed(2));
    }
  }
};
</script>

<style scoped>
.container {
  height: 100%;
  position: relative;
  box-sizing: border-box;
  border: 1px solid transparent;
  display: flex;
}
.container .control {
  float: right;
  width: 320px;
  border: 1px solid #ececec;
  margin: 10px;
  overflow-y: auto;
}
.container >>> .vux-cell-bd {
  min-width: 80px;
}
.container .charts {
  padding: 10px 0 10px 10px;
  flex: 1;
  overflow-y: auto;
}
.container .charts .chart {
  border: 1px solid #DDD;
  padding: 0;
  width: 33.3%;
  float: left;
  box-sizing: border-box;
  margin: -1px 0 0 -1px;
}
.container .charts .chart dt {
  padding: 10px;
}
.container .charts .chart dd {
  padding: 0 10px;
}
</style>
