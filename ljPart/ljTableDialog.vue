<!--
版本更新：
2020-09-01：增加ljTable两个配置属性
2020-11-02：取消open时的刷新方法，因为lj-table初始化自动会调用服务接口，不需要外部手动刷新
version: 1.0.2
-->
<template>
  <el-dialog :visible.sync="dialogVisible" :before-close="handleClose" :title="title" width="80%" :modal-append-to-body="false" :append-to-body="true" :close-on-click-modal="false">
    <lj-table
      ref="ljTable"
      :search-query ="searchQuery"
      :service-get-page="serviceGetPage"
      :data-structure="dataStructure"
      :show-pagination="showPagination"
      :show-option-column="false">
      <template slot="columns">
        <slot name="columns"/>
      </template>
    </lj-table>
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
    title: { type: String, default: '默认标题' },
    searchQuery: { type: Object, default: function() { return {} } },
    serviceGetPage: { type: Function, default: function(query) {} },
    dataStructure: { type: String, default: 'page' },
    showPagination: { type: Boolean, default: false }
  },
  data: function() {
    return {
      dialogVisible: false
    }
  },
  mounted() {
    this.$on('open', () => {
      this.openDialog()
    })
    this.$on('reload', (isResetPage) => {
      this.refreshTableData(isResetPage)
    })
  },
  methods: {
    openDialog() {
      console.log('打开弹框')
      this.dialogVisible = true
    },
    refreshTableData(isResetPage) {
      console.log('刷新表格数据', isResetPage)
      this.$refs.ljTable.$emit('table-reload', isResetPage)
    },
    handleClose(done) {
      console.log('关闭弹框前')
      done()
    }
  }
}
</script>
