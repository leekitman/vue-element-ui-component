<template>
  <el-table-column :prop="prop" :label="label" v-if="isHidden" :width="width" :formatter="formatter"/>
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
    colHidden: { type: Boolean, default: true }
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
