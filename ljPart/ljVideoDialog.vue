<!--
版本更新：
version: 1.0.0
-->
<template>
  <el-dialog :visible.sync="dialogVisible" :before-close="handleClose" :title="title" width="80%" :modal-append-to-body="false" :append-to-body="true" :close-on-click-modal="false">
    <video class="main-video" controls :src="videoPath"/>
  </el-dialog>
</template>
<script>
import ljTable from '@/components/ljPart/ljTable'
import ljTableColumn from '@/components/ljPart/ljTableColumn'
export default {
  name: 'LjTableDialog',
  components: {
    ljTable,
    ljTableColumn
  },
  props: {
    title: { type: String, default: '默认标题' }
  },
  data: function() {
    return {
      dialogVisible: false,
      videoPath: ''
    }
  },
  mounted() {
    this.$on('open', (videoPath) => {
      this.openDialog(videoPath)
    })
    this.$on('reload', (isResetPage) => {
      this.refreshTableData(isResetPage)
    })
  },
  methods: {
    openDialog(videoPath) {
      console.log('打开视频弹框', videoPath)
      this.videoPath = videoPath
      this.dialogVisible = true
    },
    handleClose(done) {
      console.log('关闭弹框前')
      done()
    }
  }
}
</script>
<style lang="scss">

.el-dialog{
    margin-top: 15vh;
    width:80%;
    height: 80vh;
    .el-dialog__body {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        .main-video {
            height: 65vh;
        }
    }
}
</style>
