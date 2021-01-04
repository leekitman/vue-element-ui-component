<!--
version: 1.0.1
2020-12-09更新：增加标题自定义的参数

标题自定义：例如将标题换行显示
<lj-table-column prop="fifthPresent" :label="abc123" :render-header="renderHeader"/>
renderHeader(h, { column, $index }) {
      return h('div', {
        attrs: {
          class: 'cell'
        },
        domProps: {
          innerHTML: '<span>' + column.label.substring(0, 3) + '</span><br/><span>' + column.label.substring(3) + '</span>'
        }
      })
    }
-->
<template>
  <el-table-column :prop="prop" :label="label" v-if="isHidden" :width="width" :formatter="formatter" :render-header="renderHeader"/>
</template>
<script>
export default {
  name: 'LjTableColumn',
  props: {
    width: { type: String, default: '' },
    prop: { type: String, default: '' },
    label: { type: String, default: '' },
    formatter: { type: Function, default: (row, column, cellValue, index) => { return cellValue } },
    colType: { type: String, default: 'normal' },
    colHidden: { type: Boolean, default: true },
    renderHeader: { type: Function, default: null }
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
  },
  watch: {
    hidden() {
      console.log('hidden改变')
      console.log(this.hidden)
    }
  }
}
</script>
