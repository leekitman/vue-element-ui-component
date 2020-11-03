<!--
示例：
<lj-textarea-column prop="intro" label="简介"/>
 -->
<template>
  <el-table-column :prop="prop" :label="label" :width="width" v-if="isHidden" :formatter="formatter">
    <template slot-scope="scope">
      <div>{{ scope.row[prop] | cutStringAtTooLongFilter(maxLenght) }}</div>
    </template>
  </el-table-column>
</template>

<script>
export default {
  name: 'StringListColumn',
  filters: {
    cutStringAtTooLongFilter(value, maxLenght) {
      if (!value) return ''
      value = value.toString()
      if (value.length > maxLenght) {
        return value.substr(0, maxLenght) + '...'
      }
      return value
    }
  },
  props: {
    prop: { type: String, default: '' },
    label: { type: String, default: '' },
    width: { type: String, default: '500' },
    formatter: { type: Function, default: (row, column, cellValue, index) => { return cellValue } },
    colType: { type: String, default: 'normal' },
    colHidden: { type: Boolean, default: true },
    maxLenght: { type: Number, default: 100 }
  },
  data() {
    return {
      hidden: this.colHidden
    }
  },
  computed: {
    isHidden() {
      if (this.colType === 'ctrl' && this.hidden) {
        return false
      } else {
        return true
      }
    }
  }
}
</script>
