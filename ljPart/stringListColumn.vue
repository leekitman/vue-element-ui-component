<!--
示例：
<string-list-column prop="titleList" label="标题列表"/>
 -->
<template>
  <el-table-column :prop="prop" :label="label" :width="width" v-if="isHidden">
    <template slot-scope="scope">
      <template v-for="str in scope.row[prop]">
        <div :key="str">{{ str }}</div>
      </template>
    </template>
  </el-table-column>
</template>

<script>
export default {
  name: 'StringListColumn',
  props: {
    prop: { type: String, default: '' },
    label: { type: String, default: '' },
    width: { type: String, default: '100' },
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
  }
}
</script>
