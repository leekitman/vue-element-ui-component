<!--
示例：
<image-popover-column prop="previewUrlFile" label="帖子预览图"/>
 -->
<template>
  <el-table-column :prop="prop" :label="label" v-if="isHidden">
    <template slot-scope="scope">
      <audio :src="scope.row[prop]" controls="controls"></audio>
    </template>
  </el-table-column>
</template>

<script>
export default {
  name: 'ImagePopoverColumn',
  props: {
    prop: { type: String, default: '' },
    label: { type: String, default: '' },
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
