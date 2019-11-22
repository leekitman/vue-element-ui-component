<!--
imageArray的json格式：
[
  "https://test-1252097922.cos.ap-guangzhou.myqcloud.com/1.jpg",
  "https://test-1252097922.cos.ap-guangzhou.myqcloud.com/2.jpg"
]
-->
<template>
  <div class="box">
    <div class="image-div" v-for="image in imageArray" :key="image">
      <el-popover
        placement="right"
        title=""
        trigger="hover">
        <img class="image-img-big" :src="image" style="max-height: 500px;max-width: 500px">
        <img class="image-img" slot="reference" :src="image"/>
      </el-popover>
    </div>
  </div>
</template>
<script>
export default {
  name: 'ImageShowBox',
  props: {
    imageArray: { type: Array, default: function() { return [] } }
  }
}
</script>
<style scoped>
.box{
  display: flex;
  flex-direction: row;
  align-items: center;
}
.image-div{
  height: 100px;
  widows: 100px;
  margin-left: 10px;
}
.image-img{
  max-height:100px;
  max-width: 100px;
}
.image-img-big{
  max-height: 500px;
  max-width: 500px;
}
</style>
