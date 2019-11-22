<!-- 树形对象下拉框 -->
<!--
version: 1.0.0
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
    <ljObjectSelectOptions v-if="options && options.length > 0" :options="options" :show-field="showField"/>
  </el-select>
</template>
<script>
import ljObjectSelectOptions from '@/components/ljPart/ljObjectSelectOptions'
export default {
  name: 'LjSelect',
  components: {
    ljObjectSelectOptions
  },
  props: {
    options: { type: Array, default: () => { return [] } },
    value: { type: Object / Array, default: function() { return {} } / function() { return [] } },
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
<style scoped>
.el-select{
    max-width: 100%;
    min-width: 300px;
}
</style>
