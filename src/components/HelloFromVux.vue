<template>
  <div class="container">
    <group title="月供">
      <cell title="房贷" value="0/月"></cell>
      <cell title="借款" value="0/月"></cell>
    </group>
    <group title="设置">
      <cell title="房价" :inline-desc="housingPrice + '万'" primary="content">
        <range v-model="housingPrice" :min="50" :max="150" :range-bar-height="4"></range>
      </cell>
      <cell title="首付" :inline-desc="downpayment + '万'" primary="content">
        <range v-model="downpayment" :range-bar-height="4"></range>
      </cell>
      <cell title="贷款" :inline-desc="loan + '万'" primary="content">
        <range v-model="loan" :range-bar-height="4"></range>
      </cell>
      <cell title="借款" :inline-desc="lend + '万'" primary="content">
        <range v-model="lend" :range-bar-height="4"></range>
      </cell>
      <cell title="贷款时长" :inline-desc="loanTerm + '年'" primary="content">
        <range v-model="loanTerm" :max="30" :range-bar-height="4"></range>
      </cell>
      <cell title="借款时长" :inline-desc="lendTerm + '年'" primary="content">
        <range v-model="lendTerm" :max="10" :range-bar-height="4"></range>
      </cell>
    </group>
    <load-more :show-loading="false" background-color="#fbf9fe"></load-more>
  </div>
</template>

<script>
import { Group, Cell, Range, LoadMore } from 'vux'

export default {
  components: {
    Group,
    Cell,
    Range,
    LoadMore
  },
  data () {
    return {
      housingPrice: 100,
      loan: 30, // 贷
      loanTerm: 15, // 贷款年限
      lend: 30, // 借
      lendTerm: 5, // 借款年限
      downpayment: 30, // 首付
      data9: 0
    }
  }
}
</script>

<style scoped>
.container >>> .vux-cell-bd {
  min-width: 80px;
}
</style>
