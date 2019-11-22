<!-- 树形表格组件 -->
<!--
数据示例：
  tableData:[{
                id: '1', // 主键PK
                title: '博客管理', // 菜单标题
                children: [{
                    id: '4',
                    title: '博客发布'
                }]
            }]

-->
<template>
  <div class="tree-table-box">
    <!-- 表格列显示控制区 -->
    <div class="filter-container">
      <el-checkbox-group ref="ctrlColCheckbox" v-model="ctrlColCheckboxVal">
        <el-checkbox v-for="checkItem in ctrlColumns" :label="checkItem.prop" :key="checkItem.prop">{{ checkItem.label }}</el-checkbox>
      </el-checkbox-group>
    </div>
    <el-table
      ref="treeTable"
      :data="tableData"
      style="width: 100%;margin-bottom: 20px;"
      row-key="id"
      border
      default-expand-all
      :tree-props="treeProps">

      <slot name="columns"/>

      <el-table-column :label="$t('table.actions')" v-if="showOptionColumn" align="center" :width="optionColWidth" class-name="small-padding fixed-width" fixed="right">
        <template slot-scope="scope">
          <el-tooltip class="item" effect="dark" content="编辑" placement="top-start">
            <el-button type="primary" icon="el-icon-edit" v-if="showEditBtn" :disabled="!showEditBtn" circle @click="clickEdit(scope.row)"/>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="删除" placement="top-start">
            <el-button type="danger" icon="el-icon-delete" v-if="showDeleteBtn" :disabled="!showDeleteBtn" circle @click="deleteTheRow(scope.row)"/>
          </el-tooltip>
          <slot name="colRightBtns" :scope="scope"/>
        </template>
      </el-table-column>
    </el-table>

  </div>
</template>

<script>
// import Vue from 'vue'
import ImagePopoverColumn from '@/components/columnFormat/imagePopoverColumn'
export default {
  name: 'LjTreeTable',
  components: {
    ImagePopoverColumn
  },
  props: {
    deleteTips: { type: String, default: '此操作会将该记录永久删除节点记录及其所有下属节点记录, 是否继续?' },
    showOptionColumn: { type: Boolean, default: true },
    showEditBtn: { type: Boolean, default: true },
    showDeleteBtn: { type: Boolean, default: true },
    optionColWidth: { type: String, default: '150' },
    serviceGetData: { type: Function, default: function() {} },
    serviceDeleteRow: { type: Function, default: function() {} },
    nodeChildrenName: { type: String, default: 'children' }
  },
  data() {
    return {
      ctrlColCheckboxVal: [],
      allCtrlColVal: [],
      ctrlColumns: [],
      treeProps: {
        children: this.nodeChildrenName,
        hasChildren: 'hasChildren'
      },
      listLoading: true,
      // 菜单表格结构数据
      tableData: []
    }
  },
  watch: {
    ctrlColCheckboxVal() {
      for (const children of this.$refs.treeTable.$children) {
        if (children.colType === 'ctrl') {
          let isTrue = true
          for (const checkboxVal of this.ctrlColCheckboxVal) {
            if (children.prop === checkboxVal) {
              children.hidden = false
              isTrue = false
            }
          }
          if (isTrue) {
            children.hidden = true
          }
        }
      }
    }
  },
  // 初始化函数，赋值，menusTree =>tableData
  created() {
    this.getTableData()
    this.getCtrlColumns()
  },
  mounted() {
    this.$on('table-reload', (isResetPage) => {
      this.getTableData()
    })
  },
  methods: {
    showRow: function(row) {
      const show = row.row.parent
        ? row.row.parent._expanded && row.row.parent._show
        : true
      row.row._show = show
      return show
        ? 'animation:treeTableShow 1s;-webkit-animation:treeTableShow 1s;'
        : 'display:none;'
    },
    levelDeep(level) {
      const distance = 15 * (level - 1) + 5
      return 'display: inline-block;width: 20px;margin-left: ' + distance + 'px;'
    },
    clickEdit(row) {
      this.$emit('column-edit-click', row)
    },
    getTableData() {
      this.listLoading = true
      this.serviceGetData().then(response => {
        this.tableData = response.data.data
        for (const row of this.tableData) {
          if (row.hasOwnProperty(this.idName)) {
            row.id = row[this.idName]
          }
        }
        this.listLoading = false
      })
    },
    getCtrlColumns() {
      const ctrlColumnsTemp = []
      const ctrlColCheckboxValTemp = []
      const allCtrlColValTemp = []
      console.log('slots列表：', this.$slots)
      for (const column of this.$slots.columns) {
        if (column.componentOptions && column.componentOptions.propsData.colType === 'ctrl') {
          const propsData = column.componentOptions.propsData
          ctrlColumnsTemp.push({
            'label': propsData.label,
            'prop': propsData.prop
          })
          allCtrlColValTemp.push(propsData.prop)
          if (propsData.colHidden !== undefined && !propsData.colHidden) {
            ctrlColCheckboxValTemp.push(propsData.prop)
          }
        }
      }
      this.ctrlColumns = ctrlColumnsTemp
      this.ctrlColCheckboxVal = ctrlColCheckboxValTemp
      this.allCtrlColVal = allCtrlColValTemp
    },
    deleteTheRow(row) {
      console.log('删除时row')
      console.log(row)
      this.$confirm(this.deleteTips, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 删除
        this.serviceDeleteRow(row.id).then(response => {
          this.$message(response.data.message)
          this.getTableData()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style scoped>

</style>
