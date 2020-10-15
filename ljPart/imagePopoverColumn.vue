<!--
示例：
<image-popover-column prop="previewUrlFile" label="帖子预览图"/>
 -->
<template>
  <el-table-column :prop="prop" :label="label" width="75" v-if="isHidden">
    <template slot-scope="scope">
      <image-popover :image-url="scope.row[prop]"/>
    </template>
  </el-table-column>
</template>

<script>
import ImagePopover from '@/components/columnFormat/imagePopover'
export default {
  name: 'ImagePopoverColumn',
  components: {
    ImagePopover
  },
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
