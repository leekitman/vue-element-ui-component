<template>
  <el-popover
    placement="right"
    title=""
    trigger="hover">
    <img :src="imageUrl" style="max-height: 500px;max-width: 500px">
    <img slot="reference" :src="imageUrl" alt="" style="width: 50px;height: 50px">
  </el-popover>
</template>

<script>
export default {
  name: 'ImagePopover',
  props: {
    imageUrl: { type: [String, Array], default: '' }
  }
}
// 示例： <image-popover :image-url="scope.row.resourcePath"/>
</script>
