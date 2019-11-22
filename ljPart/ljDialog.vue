<!--
props说明：
序号	props属性名	        类型	      作用描述	                                            默认值	示例
1	    titleSuffix	      String	    对话框的标题后缀，比如【添加相册】、【编辑相册】中的相册	  ""
2	    form	            Object	    form表单的数据对象	                                    {}	请查看数据格式-form
3	    rules	            Object	    form表单的校验规则	                                    []	请查看数据格式-rules
4	    serviceRefresh	  Function	  表单提交成功调用的方法，一般用于重载数据表格	              function() {}
5	    serviceGetInfo	  Function	  根据id获取一条记录的api方法	                              function() {}
6	    serviceAddRow	    Function	  添加记录的api方法	function() {}
7	    serviceUpdateRow	Function	  更新记录的api方法	function() {}
8	    labelWidth	      String	    左边label的宽度	100px
9	    width	            String	    整个对话框占父元素的百分比	50%
10    beforeSubmitFun   Function    提交之前执行的方法，比如要在提交前把富文本编辑器的内容赋值到form对象

触发事件说明：
序号	事件名	     参数	  触发后的作用	用法示例
1	    open		           打开空白对话框	this.$refs.clubsDialog.$emit('open')
2	    open-update	row	   打开空白对话框并请求数据回显。row是一个必须有非空的id属性的对象	this.$refs.clubsDialog.$emit('open-update', row)
3     reload              刷新回显数据，可以解决级联下拉框组件不回显问题

绑定事件说明：
序号	事件名	参数	触发后的作用	用法示例
1	reset-form		重置表单，在关闭对话框，提交表单成功时触发	@reset-form="resetForm"，双引号中是自定义方法。

数据格式
1、rules：
{
 "code": [{"required": true,"message": "请输入唯一code","trigger": "blur"}],
 "originalAuthor": [{"required": true,"message": "请输入相册名称","trigger": "blur"}]
}

1、form：
{id:"",code:"",originalAuthor:""}

更新说明：
2019-06-10：<el-form>标签增加了v-if指令，为了避免上图文件组件在上传一次后关闭弹框再打开不可上传的问题，所以强制重新渲染
2019-08-16：loading在提交表单请求失败时一直为true，导致下次打开弹框也显示loading
-->
<template>
  <div>
    <!-- 表单对话框 -->
    <el-dialog :visible.sync="dialogVisible" :before-close="handleClose" :title="dialogTitle" :width="width" :modal-append-to-body="false" :append-to-body="true" :close-on-click-modal="false" v-loading="loading">
      <el-form ref="form" :rules="rules" :model="form" status-icon :label-width="labelWidth" v-if="dialogVisible">

        <slot/>

      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="cancelForm">取 消</el-button>
        <el-button type="primary" @click="submitForm">提 交</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  props: {
    titleSuffix: { type: String, default: '' },
    form: { type: Object, default: () => { return {} } },
    rules: { type: Object, default: () => { return {} } },
    serviceRefresh: { type: Function, default: () => {} },
    serviceGetInfo: { type: Function, default: (id) => {} },
    serviceAddRow: { type: Function, default: () => {} },
    serviceUpdateRow: { type: Function, default: () => {} },
    labelWidth: { type: String, default: '100px' },
    width: { type: String, default: '50%' },
    beforeSubmitFun: { type: Function, default: () => {} }
  },
  data() {
    return {
      dialogVisible: false,
      dialogCurrentOption: '添加',
      dialogCurrentMethod: 'post',
      loading: false
    }
  },
  computed: {
    dialogTitle: function() {
      return this.dialogCurrentOption + this.titleSuffix
    }
  },
  mounted() {
    this.$on('open', () => {
      this.openDialog()
    })
    this.$on('open-update', (rowId) => {
      this.openDialogUpdate(rowId)
    })
    this.$on('reload', () => {
      this.openDialogUpdate(this.form.id)
    })
  },
  methods: {
    openDialog() {
      this.dialogCurrentOption = '添加'
      this.dialogCurrentMethod = 'post'
      this.dialogVisible = true
    },
    openDialogUpdate(rowId) {
      // 请求数据
      this.form.id = rowId
      this.serviceGetInfo(rowId).then(response => {
        // 回显到dialog
        var data = response.data.data
        for (const key in this.form) {
          if (data.hasOwnProperty(key)) {
            this.form[key] = data[key]
          }
        }
        this.$emit('reload-success', response)
      })
      this.dialogCurrentOption = '编辑'
      this.dialogCurrentMethod = 'put'
      this.dialogVisible = true
    },
    handleClose(done) {
      this.resetForm()
      done()
    },
    resetForm() {
      this.$refs.form.clearValidate()
      this.$emit('reset-form')
    },
    cancelForm() {
      this.dialogVisible = false
      this.resetForm()
    },
    submitForm() {
      this.beforeSubmitFun()
      console.log('提交表单时form')
      console.log(this.form)

      this.$refs['form'].validate((valid) => {
        if (valid) {
          console.log('提交表单,当前操作是=' + this.dialogCurrentMethod)
          if (this.dialogCurrentMethod === 'post') {
            // 添加
            console.log(this.form, 'post提交表单前的form')
            this.loading = true
            this.serviceAddRow(this.form).then(response => {
              this.loading = false
              this.$message(response.data.message)
              this.resetForm()
              this.dialogVisible = false
              this.serviceRefresh()
            }).catch(reject => {
              console.error('提交表单失败')
              this.loading = false
            })
          } else if (this.dialogCurrentMethod === 'put') {
            if (this.form.id) {
            // 编辑
              this.loading = true
              this.serviceUpdateRow(this.form).then(response => {
                this.loading = false
                this.$message(response.data.message)
                this.resetForm()
                this.dialogVisible = false
                this.serviceRefresh()
              }).catch(reject => {
                console.error('提交表单失败')
                this.loading = false
              })
            } else {
              this.$message('form.id不能为空')
            }
          }
        } else {
          console.log('表单未通过检验!!')
          return false
        }
      })
    }
  }
}
</script>
