<template>
  <div>
    <el-input type="text" v-model="theValue"/>
  </div>
</template>
<script>
export default {
  name: 'LjMoneyInput',
  props: {
    value: { type: Number / String, default: '0.00' }
  },
  data: function() {
    return {
      resultValue: ''
    }
  },
  computed: {
    theValue: {
      get: function() {
        var temValue = '0.00'
        if (this.value) {
          temValue = this.value + ''
        }
        var number = temValue.split('.')
        var prefix = number[0]
        var posfix = number[1]
        // 处理整数部分
        var result = this.prefixHandle(prefix)
        // 处理小数部分
        posfix = this.postfixHandle(posfix)
        // 整数和小数拼接
        result = result + prefix.substring(prefix.length - 3) + '.' + posfix
        return result
      },
      set: function(value) {
        var substr = new RegExp(/[^0-9\.]/, 'g')
        var result = value.replace(substr, '')
        var splitStr = result.split('.')
        if (splitStr.length > 1) {
          var prefix = splitStr[0]
          var posfix = splitStr[1]
          if (!prefix || prefix.length === 0) {
            prefix = '0'
          }
          posfix = this.postfixHandle(posfix)
          result = prefix + '.' + posfix
        }
        this.resultValue = result
        this.$emit('input', this.resultValue)
      }
    }
  },
  methods: {
    prefixHandle(prefix) {
      var mo = prefix.length % 3
      var am = prefix.length % 3 > 0 ? parseInt(prefix.length / 3) : parseInt(prefix.length / 3 - 1)
      var result = ''
      for (var i = 0; i < am; i++) {
        if (i === 0) {
          result = result + prefix.substring(0, mo) + ','
        } else {
          const subStart = i - 1 === 0 ? mo : mo + (i * 3)
          const subEnd = subStart + 3
          var subStr = prefix.substring(subStart, subEnd)
          result = result + subStr + ','
        }
      }
      return result
    },
    postfixHandle(posfix) {
      if (!posfix || posfix.length === 0) {
        posfix = '00'
      } else if (posfix.length === 1) {
        posfix = posfix + '0'
      } else if (posfix.length > 2) {
        posfix = posfix.substring(0, 2)
      }
      return posfix
    }
  }
}
</script>
