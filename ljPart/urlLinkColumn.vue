<!--
示例：
<url-link-column prop="source" label="来源网址"/>
 -->
<template>
  <el-table-column :prop="prop" :label="label" :width="width" v-if="isHidden">
    <template slot-scope="scope">
      <a :href="scope.row[prop]" v-text="scope.row[prop]" target="blank"></a>
    </template>
  </el-table-column>
</template>

<script>
export default {
  name: 'UrlLinkColumn',
  props: {
    prop: { type: String, default: '' },
    label: { type: String, default: '' },
    width: { type: String, default: '' },
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
