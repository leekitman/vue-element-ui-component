<!--
version: 1.1.4
2019-07-16更新：table-reload触发刷新时，强制重置页码为第一页，因为这样更合理
2020-02-16更新：删除成功的提示信息修改，之前一直是空的
2020-02-17更新：刷新列表，页数不变
2020-02-17更新：table增加row-key属性，用于兼容树形表格
2020-02-21更新：table刷新时根据参数决定是否重置页数
props说明：
序号	props属性名	        类型	    作用描述	                                                              默认值
1	    deleteTips	      String	    删除按钮点击后的提示内容	                                              “此操作会将该记录永久删除, 是否继续?”
2	    searchQuery	      Object	    表格数据重载时的参数，用于带参刷新和搜索	                               {}
4	    showEditBtn	      Boolean	    行末【编辑】按钮是否隐藏和禁用	                                        true
5	    showDeleteBtn	    Boolean	    行末【删除】按钮是否隐藏和禁用	                                        true
6	    showOptionColumn	Boolean	    行末操作列是否隐藏	                                                   true
7	    showPagination	  Boolean	    分页组件是否显示	                                                     true
8	    serviceGetPage	  Function	  获取表格数据api方法	                                                   function() {}
9	    serviceDeleteRow	Function	  删除表格记录api方法                                                    function() {}
10	  idName	          String	    表格数据中主键的属性名	                                                id
11	  optionColWidth	  String	    行末操作列的列宽	                                                     150
12	  selectionChange	  Function	  表格多选触发并调用该函数传递当前选中行	                                 function(sels) {}
13	  dataStructure	    String	    表格数据请求的响应数据结构类型，page是分页结构，data是直接返回的对象	     page

触发事件说明：
序号	    事件名	                参数	        触发后的作用	                                                                用法示例
1	        table-reload		       isResetPage   重载表格数据，若props属性searchQuery有值，则会带参重载；参数代表是否重置页码	     this.refs.clubsTable.emit('table-reload',true)

绑定事件说明：
序号	    事件名	                参数	        触发后的作用	                                                                用法示例
1	        column-edit-click	     row	        组件内行末【编辑】按钮点击后触发，将调用父组件绑定的方法；一般用于打开对话框	      @column-edit-click="openDialogUpdate"，双引号中是自定义方法。

数据格式
1、searchQuery：
{"code":"a","name":"a","unitName":"a"}

2、otherColumn：
[{"label":"正文图片","prop":"resourcePath","component":"ImagePopoverColumn"}]
label：列名
prop：字段名
component：指定格式化组件；组件必须有label和prop两个属性，因为其分别用于绑定上面的label和prop

增加末列操作按钮示例：
<lj-table>
  <template slot="colRightBtns" slot-scope="option">
    <el-tooltip class="item" effect="dark" content="排序" placement="top-start">
      <el-button type="primary" @click="sort(option.scope.row)" circle>排</el-button>
    </el-tooltip>
  </template>
</lj-table>
-->
<template>
  <div>
    <!-- 表格列显示控制区 -->
    <div class="filter-container">
      <el-checkbox-group ref="ctrlColCheckbox" v-model="ctrlColCheckboxVal">
        <el-checkbox v-for="checkItem in ctrlColumns" :label="checkItem.prop" :key="checkItem.prop">{{ checkItem.label }}</el-checkbox>
      </el-checkbox-group>
    </div>

    <!-- 表格 -->
    <el-table
      ref="elTable"
      v-loading="listLoading"
      :data="tableData"
      :key="key"
      row-key="id"
      border
      fit
      highlight-current-row
      style="width: 100%"
      @selection-change="selsChange"
      @current-change="currentChangeFun">
      <el-table-column type="index" width="30"/>
      <el-table-column type="selection" width="40" v-if="showSelectionColumn"/>

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

    <!-- 分页组件 -->
    <div class="pagination-container" v-if="showPagination">
      <el-pagination :current-page="listQuery.pageNo" :page-sizes="[10,20,40,60]" :page-size="listQuery.pageSize" :total="total" background layout="total, sizes, prev, pager, next, jumper" @size-change="handleSizeChange" @current-change="handleCurrentChange" :hide-on-single-page="hideOnSinglePage"/>
    </div>
  </div>
</template>

<script>
import ImagePopoverColumn from '@/components/columnFormat/imagePopoverColumn'

export default {
  components: {
    ImagePopoverColumn
  },
  props: {
    deleteTips: { type: String, default: '此操作会将该记录永久删除, 是否继续?' },
    searchQuery: { type: Object, default: () => { return {} } },
    showEditBtn: { type: Boolean, default: true },
    showDeleteBtn: { type: Boolean, default: true },
    showOptionColumn: { type: Boolean, default: true },
    showSelectionColumn: { type: Boolean, default: false },
    showPagination: { type: Boolean, default: false },
    serviceGetPage: { type: Function, default: function() {} },
    serviceDeleteRow: { type: Function, default: function() {} },
    idName: { type: String, default: 'id' },
    optionColWidth: { type: String, default: '150' },
    selectionChange: { type: Function, default: function(sels) {} },
    currentChange: { type: Function, default: function(currentRow, oldCurrentRow) {} },
    dataStructure: { type: String, default: 'page' },
    hideOnSinglePage: { type: Boolean, default: false }
  },
  data() {
    return {
      ctrlColCheckboxVal: [],
      allCtrlColVal: [],
      ctrlColumns: [],
      listQuery: {
        pageNo: 1,
        pageSize: 10
      },
      listLoading: true,
      tableData: [],
      total: 0,
      key: 1 // table key
    }
  },
  watch: {
    ctrlColCheckboxVal() {
      for (const children of this.$refs.elTable.$children) {
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
  created() {
    this.listQuery = {
      pageNo: 1,
      pageSize: 10
    }
    this.getTableData()
    this.getCtrlColumns()
  },
  mounted() {
    this.$on('table-reload', (isResetPage) => {
      if (isResetPage) {
        this.listQuery = {
          pageNo: 1,
          pageSize: 10
        }
      }
      this.getTableData()
    })
  },
  methods: {
    getTableData() {
      this.listLoading = true
      var query
      if (!this.showPagination) {
        query = this.searchQuery
      } else {
        query = Object.assign(this.searchQuery || {}, this.listQuery)
      }
      this.serviceGetPage(query).then(response => {
        console.log('加载完成中:response')
        console.log(response)
        if (this.dataStructure === 'page') {
          this.tableData = response.data.data.result
        } else if (this.dataStructure === 'data') {
          this.tableData = response.data.data
        }
        console.log('主键名:' + this.idName)
        for (const row of this.tableData) {
          if (row.hasOwnProperty(this.idName)) {
            row.id = row[this.idName]
          }
        }
        console.log('数据加载完成后，tableData')
        console.log(this.tableData)
        this.total = parseInt(response.data.data.totalCount)
        this.listLoading = false
      })
    },
    getCtrlColumns() {
      const ctrlColumnsTemp = []
      const ctrlColCheckboxValTemp = []
      const allCtrlColValTemp = []
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
    handleSizeChange(val) {
      this.listQuery.pageSize = val
      this.getTableData()
    },
    handleCurrentChange(val) {
      this.listQuery.pageNo = val
      this.getTableData()
    },
    clickEdit(row) {
      this.$emit('column-edit-click', row)
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
          this.$message({
            type: 'success',
            message: '删除成功'
          })
          this.getTableData()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    selsChange(sels) {
      this.selectionChange(sels)
    },
    currentChangeFun(currentRow, oldCurrentRow) {
      this.currentChange(currentRow, oldCurrentRow)
    }
  }
}
</script>
