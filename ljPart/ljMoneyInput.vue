<!--
version: 1.1.0
2020-03-17更新：使输入体验更佳
2020-10-27更新：增加最大值最小值限制
-->
<template>
  <div>
    <el-input type="number" v-model="theValue" @blur="onBlurFun" step="0.01" :min="min" :max="max"/>
  </div>
</template>
<script>
export default {
  name: 'LjMoneyInput',
  props: {
    value: { type: Number / String, default: '0.00' },
    min: { type: Number / String, default: null },
    max: { type: Number / String, default: null }
  },
  data: function() {
    return {
      resultValue: '0.00'
    }
  },
  computed: {
    theValue: {
      get: function() {
        return this.value ? this.value : '0.00'
      },
      set: function(value) {
        this.resultValue = value
        this.$emit('input', this.resultValue)
      }
    }
  },
  methods: {
    postfixHandle(posfix) {
      if (!posfix || posfix.length === 0) {
        posfix = '00'
      } else if (posfix.length === 1) {
        posfix = posfix + '0'
      } else if (posfix.length > 2) {
        posfix = posfix.substring(0, 2)
      }
      return posfix
    },
    onBlurFun(one, two, three, four) {
      var value = this.resultValue
      var substr = new RegExp(/[^0-9\.]/, 'g')
      var result = value.replace(substr, '')
      var splitStr = result.split('.')
      var prefix = splitStr[0]
      if (!prefix || prefix.length === 0) {
        prefix = '0'
      }
      var posfix = splitStr[1]
      if (!posfix || posfix.length === 0) {
        posfix = '00'
      }
      posfix = this.postfixHandle(posfix)
      result = prefix + '.' + posfix
      this.resultValue = result
      this.$emit('input', this.resultValue)
    }
  }
}
</script>
