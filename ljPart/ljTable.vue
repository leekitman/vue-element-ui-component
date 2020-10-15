<!--
version: 1.1.11
2019-07-16更新：table-reload触发刷新时，强制重置页码为第一页，因为这样更合理
2020-02-16更新：删除成功的提示信息修改，之前一直是空的
2020-02-17更新：刷新列表，页数不变
2020-02-17更新：table增加row-key属性，用于兼容树形表格
2020-02-21更新：table刷新时根据参数决定是否重置页数
2020-04-09更新：增加外部控制loding的事件
2020-09-29更新：增加defaultExpandAll属性
                序号列名称默认“序号”
2020-09-29更新：增加selectable属性：决定每一行是否可选
2020-10-12更新：增加组件doc注释
2020-10-15更新：增加after-refresh绑定事件，用于传递表格数据给父组件
2020-10-15更新：增加rowClassName，设置每一行的样式
2020-10-15更新：增加height，设置表格高度

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
      :height="height?height:false"
      row-key="id"
      border
      fit
      highlight-current-row
      style="width: 100%"
      :default-expand-all="defaultExpandAll"
      :row-class-name="rowClassName"
      @selection-change="selsChange"
      @current-change="currentChangeFun">
      <el-table-column type="index" width="50" label="序号"/>
      <el-table-column type="selection" width="45" v-if="showSelectionColumn" :selectable="selectable"/>

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
/**
 * 分页数据表格
 * @description 统一风格的数据表格，简化了许多功能
 * @property {String} deleteTips 点击删除按钮弹出确认框的提示内容
 * @property {Object} searchQuery 触发事件`table-reload`，组件会调用`serviceGetPage`方法刷新表格数据，方法的参数就是`searchQuery`，数据格式：{"code":"a","name":"a","unitName":"a"}
 * @property {Boolean} showEditBtn 是否显示右侧操作栏的【编辑】按钮
 * @property {Boolean} showDeleteBtn 是否显示右侧操作栏的【删除】按钮
 * @property {Boolean} showOptionColumn 是否显示右侧操作栏
 * @property {Boolean} showSelectionColumn 是否显示左侧多选栏
 * @property {Boolean} showPagination 是否显示底部分页组件
 * @property {Function} serviceGetPage 服务接口-获取表格数据
 * @property {Function} serviceDeleteRow 服务接口-删除一条表格数据
 * @property {String} idName 服务接口`serviceGetPage`的结果中，哪个字段作为表格数据主键。默认'id'
 * @property {String} optionColWidth 右侧操作栏宽度，默认150
 * @property {Function} selectionChange 当表格多选发生变更时会被组件调用的回调函数
 * @property {Function} currentChange 当表格当选发生变更时会被组件调用的回调函数
 * @property {String} dataStructure 服务接口`serviceGetPage`的请求结果是什么结构类型：可选项[page, data]
 * @property {Boolean} hideOnSinglePage 是否在只有一页数据时隐藏分页组件
 * @property {Boolean} defaultExpandAll 是否默认展开所有节点
 * @property {Function} selectable 判断每一行是否可选
 * @property {Function} rowClassName 判断每一行的样式，主要用于设置不同状态的颜色
 *
 * 触发事件：用法示例：this.refs.clubsTable.emit('table-reload',true)
 * @event  table-reload(isResetPage)  重载表格数据，若props属性searchQuery有值，则会带参重载；参数代表是否重置页码
 * 
 * 绑定事件：用法示例：@column-edit-click="openDialogUpdate"
 * @event  column-edit-click(row)     组件内行末【编辑】按钮点击后触发，将调用父组件绑定的方法；一般用于打开对话框
 * @event  after-refresh(tableData)   表格数据刷新后触发，将调用父组件绑定的方法；一般用于传递表格数据
 */
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
    hideOnSinglePage: { type: Boolean, default: false },
    defaultExpandAll: { type: Boolean, default: false},
    selectable: { type: Function, default: function(row, index) {return true} },
    rowClassName: { type: Function, default: function({row, index}) {return true} },
    height: { type: String|Number, default: '' },
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
    this.$on('loading-show', () => {
      this.listLoading = true
    })
    this.$on('loading-hidden', () => {
      this.listLoading = false
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
        this.$emit('after-refresh', this.tableData)
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
