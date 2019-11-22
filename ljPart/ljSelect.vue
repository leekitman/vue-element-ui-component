<!-- 树形下拉框 -->
<!--
version: 1.0.1
更新日期：2019-07-18
更新内容：增加props到v-model之间的计算属性，解决回显失败的问题
options数据结构：
[
    {
        "id": "1",
        "text": "一号"，
        "children": {
            "id": "12",
            "text": "一二号"
        }
    }
]
-->
<template>
  <el-select v-model="miValue" @input="onChange" placeholder="请选择" :clearable="true" :disabled="disabled" filterable :multiple="multiple">
    <ljSelectOptions v-if="options && options.length > 0" :options="options" :show-field="showField"/>
  </el-select>
</template>
<script>
import ljSelectOptions from '@/components/ljPart/ljSelectOptions'
export default {
  name: 'LjSelect',
  components: {
    ljSelectOptions
  },
  props: {
    options: { type: Array, default: () => { return [] } },
    value: { type: String / Array, default: '' / function() { return [] } },
    disabled: { type: Boolean, default: false },
    showField: { type: String, default: 'text' },
    multiple: { type: Boolean, default: false }
  },
  data() {
    return {
      otherValue: this.value
    }
  },
  computed: {
    miValue: {
      get: function() {
        return this.value
      },
      set: function(newValue) {
        this.otherValue = newValue
      }
    }
  },
  methods: {
    onChange(value) {
      this.$emit('input', this.otherValue)
    }
  }
}
</script>
