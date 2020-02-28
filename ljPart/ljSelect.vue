<!-- 树形下拉框 -->
<!--
version: 1.0.2
更新日期：2019-07-18
更新内容：增加props到v-model之间的计算属性，解决回显失败的问题
更新2020-02-27：增加远程数据懒加载树机制
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
  <el-select :value="value" @input="onChange" :value-key="valueKey" :placeholder="placeholder" :clearable="true" :disabled="disabled" filterable :multiple="multiple">
    <lj-select-options
      v-if="(local && options && options.length > 0) || !local"
      :show-field="showField"
      :expand-all="expandAll"
      :value-key="valueKey"
      :value-object="valueObject"
      :local="local"
      :local-data="options"
      :load-parent-id="loadRootParentId"
      :load="load"/>
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
    value: { type: String / Object / Array, default: '' / function() { return {} } / function() { return [] } },
    disabled: { type: Boolean, default: false },
    valueKey: { type: String, default: 'id' },
    showField: { type: String, default: 'text' },
    multiple: { type: Boolean, default: false },
    valueObject: { type: Boolean, default: false },
    expandAll: { type: Boolean, default: true },
    placeholder: { type: String, default: '请选择' },
    // 本地数据配置
    local: { type: Boolean, default: true },
    options: { type: Array, default: () => { return [] } },
    // 远程数据配置
    loadRootParentId: { type: String, default: '0' },
    load: { type: Function, default: function(node, resolve) {} }
  },
  data() {
    return {
    }
  },
  methods: {
    onChange(value) {
      this.$emit('input', value)
    }
  }
}
</script>
