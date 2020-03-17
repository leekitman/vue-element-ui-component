<!--
version: 1.0.1
2020-03-17更新：刷新外部表格时，增加是否重置页码的值
示例：
<lj-switch-column prop="status" label="状态" false-text="禁用" true-text="启用"/>
或
<lj-switch-column
          prop="status"
          label="状态"
          false-text="禁用"
          false-value="DISABLED"
          true-text="启用"
          true-value="NORMAL"
          width="150"
          :read-only="false"
          :service-switch="serviceSwitchKeyStatus"
          :refresh-table="refreshTableData"/>
 -->
<template>
  <el-table-column :prop="prop" :label="label" :width="width" v-if="isHidden" :loading="loading">
    <template slot-scope="scope">
      <el-switch
        style="display: block"
        :value="scope.row[prop]"
        active-color="#13ce66"
        inactive-color="#ff4949"
        :active-text="trueText"
        :inactive-text="falseText"
        :active-value="trueValue"
        :inactive-value="falseValue"
        :read-only="readOnly"
        @change="onChangeFun(scope.row)">
      </el-switch>
    </template>
  </el-table-column>
</template>

<script>
import { Message } from 'element-ui'
export default {
  name: 'LjSwitchColumn',
  props: {
    prop: { type: String, default: '' },
    width: { type: String, default: '100' },
    label: { type: String, default: '' },
    colType: { type: String, default: 'normal' },
    colHidden: { type: Boolean, default: true },
    falseText: { type: String, default: 'false' },
    falseValue: { type: String / Boolean, default: false },
    trueText: { type: String, default: 'true' },
    trueValue: { type: String / Boolean, default: true },
    readOnly: { type: Boolean, default: true },
    idFieldName: { type: String, default: 'id' },
    serviceSwitch: { type: Function, default: function(id, status) {} },
    refreshTable: { type: Function, default: function(isResetPage) {} }
  },
  data() {
    return {
      hidden: this.colHidden,
      loading: false
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
  },
  methods: {
    onChangeFun(row) {
      if (this.readOnly) {
        return
      }
      this.loading = true
      console.log('切换switch', row)
      if (this.serviceSwitch) {
        var targetStatus = row[this.prop]
        switch (targetStatus) {
          case this.falseValue: targetStatus = this.trueValue
            break
          case this.trueValue: targetStatus = this.falseValue
            break
          default: Message({
            message: '状态值不匹配：' + targetStatus,
            type: 'error',
            duration: 5 * 1000
          })
            return
        }
        this.serviceSwitch(row[this.idFieldName], targetStatus)
          .then(response => {
            // this.$message(response.data.message)
            this.refreshTable(true)
          }).catch(reject => {
            console.error('状态切换失败')
            this.loading = false
          })
      } else {
        Message({
          message: 'switch组件缺少服务api',
          type: 'error',
          duration: 5 * 1000
        })
      }
    }
  }
}
</script>
