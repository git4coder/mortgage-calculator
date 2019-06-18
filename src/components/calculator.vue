<template>
  <div class="container">
    <group title="月供(等额本息)">
      <cell title="房贷" :value="loanPreMonth + '元/月'"></cell>
      <cell title="借款" :value="lendPreMonth + '元/月'"></cell>
    </group>
    <group title="设置">
      <cell title="房价" :inline-desc="housingPrice + '万'" primary="content">
        <range v-model="housingPrice" :min="0" :max="150" :range-bar-height="rangeBarHeight"></range>
      </cell>
      <cell title="首付" :inline-desc="downpayment + '万'" primary="content">
        <range v-model="downpayment" :range-bar-height="rangeBarHeight"></range>
      </cell>
      <cell title="贷款" :inline-desc="loan + '万'" primary="content">
        <range v-model="loan" :range-bar-height="rangeBarHeight"></range>
      </cell>
      <cell title="年利率" :inline-desc="loanRate + '%'" v-if="loan" primary="content">
        <inline-x-number v-model="loanRate" :step="0.01" :min="0" :max="10" fillable></inline-x-number>
      </cell>
      <cell title="贷款时长" :inline-desc="loanTerm + '年'" v-if="loan" primary="content">
        <range v-model="loanTerm" :min="1" :max="30" :range-bar-height="rangeBarHeight"></range>
      </cell>
      <cell title="借款" :inline-desc="lend + '万'" primary="content">
        <range v-model="lend" :range-bar-height="rangeBarHeight" :max="housingPrice"></range>
      </cell>
      <cell title="借款时长" :inline-desc="lendTerm + '年'" v-if="lend" primary="content">
        <range v-model="lendTerm" :min="1" :max="10" :range-bar-height="rangeBarHeight"></range>
      </cell>
    </group>
    <group title="其它">
      <cell title="利息" :value="loanInterest + '元'"></cell>
    </group>
    <load-more :show-loading="false" background-color="#fbf9fe"></load-more>
  </div>
</template>

<script>
import { Group, Cell, Range, InlineXNumber, LoadMore } from 'vux';

export default {
  components: {
    Group,
    Cell,
    Range,
    InlineXNumber,
    LoadMore
  },
  data() {
    return {
      housingPrice: 100, // 房价
      loanTerm: 15, // 贷款年限
      loanRate: 4.90, // 贷款利率
      lend: 12, // 借
      lendTerm: 5, // 借款年限
      downpayment: 30, // 首付
      rangeBarHeight: 2
    };
  },
  computed: {
    // 贷款
    loan: {
      get: function() {
        var that = this;
        var housingPrice = parseInt(that.housingPrice);
        var downpayment = parseInt(that.downpayment);
        return housingPrice - downpayment;
      },
      set: function(newValue) {
        var that = this;
        var housingPrice = parseInt(that.housingPrice);
        var value = housingPrice - newValue;
        that.downpayment = value;
        console.log('loan set:', value)
      }
    },
    // 每月应还借款
    lendPreMonth() {
      var that = this;
      var value = 0;
      var lend = parseInt(that.lend);
      var lendTerm = parseInt(that.lendTerm);
      value = (lend * 10000) / (lendTerm * 12);
      return value.toFixed(2);
    },
    // 每月应还贷款
    loanPreMonth() {
      // var that = this;
      // var loan = parseInt(that.loan);
      // var loanInterest = that.loanInterest;
      // var loanTerm = parseInt(that.loanTerm);
      // var value = (loan * 10000 + loanInterest) / loanTerm / 12;
      // return value.toFixed(2);

      var that = this;
      var loan = parseInt(that.loan);
      var loanTerm = parseInt(that.loanTerm);
      var monthRate = parseFloat(that.loanRate) / 100 / 12;
      var pow = Math.pow(1 + monthRate, loanTerm * 12);
      var value = monthRate * pow / (pow - 1) * (loan * 10000); // https://baike.baidu.com/item/等额本息/3227456
      return value.toFixed(2);
    },
    // 贷款利息
    loanInterest() {
      // var that = this;
      // var loan = parseInt(that.loan);
      // var loanRate = parseFloat(that.loanRate);
      // var loanTerm = parseInt(that.loanTerm);
      // var value = loan * 10000 * loanRate / 100 * loanTerm;
      // value = parseFloat(value.toFixed(2));
      // return value;

      var that = this;
      var loan = parseInt(that.loan) * 10000;
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
    }
  }
};
</script>

<style scoped>
.container >>> .vux-cell-bd {
  min-width: 80px;
}
</style>
