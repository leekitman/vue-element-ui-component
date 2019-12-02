<template>
  <div>
    <el-dialog
      ref="showDialog"
      :title="title"
      :visible.sync="showDialogOpen"
      :width="width"
      :modal-append-to-body="false"
      :append-to-body="true"
      :close-on-click-modal="false"
      v-loading="loading"
      :before-close="handleClose"
    >
      <div class="image-box">
        <div
          class="image-item"
          v-for="item in spiderData"
          :key="item.key">
          <div>
            <el-image
              class="image-content"
              :src="item.fullPath"
              :preview-src-list="previewImages">
            </el-image>
          </div>
          <div v-if="showDeleteBtn">
            <i class="el-icon-delete" @click="deleteImage(item.id)"></i>
          </div>
        </div>
        <div v-if="showAddBtn" class="image-item add-image-button" @click="openAddImage">
          <i class="el-icon-plus"></i>
        </div>
      </div>
    </el-dialog>
    <el-dialog
      ref="addDialog"
      title="添加图片"
      :visible.sync="addDialogOpen"
      :width="width"
      :modal-append-to-body="false"
      :append-to-body="true"
      :close-on-click-modal="false"
      v-loading="addLoading"
    >
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="添加图片" prop="images">
          <ljFileUpload v-model="form.images" multiple/>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">提 交</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import ljFileUpload from '@/components/ljPart/ljFileUpload'
export default {
  name: 'LjImageManager',
  components: {
    ljFileUpload
  },
  props: {
    width: { type: String, default: '50%' },
    title: { type: String, default: '图片列表' },
    showAddBtn: { type: Boolean, default: false },
    showDeleteBtn: { type: Boolean, default: false },
    serviceGetList: { type: Function, default: function(row) {} },
    serviceAddRow: { type: Function, default: function(data) {} },
    serviceDeleteRow: { type: Function, default: function(id) {} }
  },
  data() {
    return {
      spiderData: [],
      showDialogOpen: false,
      addDialogOpen: false,
      currentRow: '',
      form: this.initForm(),
      rules: {
        images: [{ required: true, message: '此项必填', trigger: 'blur' }]
      },
      loading: false,
      addLoading: false
    }
  },
  computed: {
    previewImages() {
      return this.spiderData.map(data => data.fullPath)
    }
  },
  mounted() {
    this.$on('open', (row) => {
      console.log('参数是：', row)
      this.openDialog(row)
    })
  },
  created() {

  },
  methods: {
    refreshImageList() {
      // 获取店铺轮播图
      this.loading = true
      this.serviceGetList(this.currentRow).then(res => {
        console.log('新增上传图片成功', res)
        this.spiderData = res.data.data
        this.loading = false
      }).catch(error => {
        console.error('新增上传图片失败', error)
        this.loading = false
      })
    },
    openDialog(row) {
      console.log('打开窗口', row)
      this.currentRow = row
      this.showDialogOpen = true
      this.refreshImageList()
    },
    handleClose(done) {
      this.spiderData = []
      done()
    },
    initForm() {
      return {
        productId: '',
        images: null,
        resourceType: 'SLIDESHOW'
      }
    },
    resetForm() {
      this.form = this.initForm()
    },
    openAddImage() {
      console.log('打开图片上传组件', this.currentRow)
      this.form.productId = this.currentRow.id
      this.addDialogOpen = true
    },
    submitForm() {
      console.log('提交上传图片表单', this.form)
      this.addLoading = true
      this.serviceAddRow(this.form).then(response => {
        this.addLoading = false
        this.$message(response.data.message)
        this.resetForm()
        this.addDialogOpen = false
        this.refreshImageList()
      }).catch(reject => {
        console.error('提交表单失败', reject)
        this.addLoading = false
      })
    },
    deleteImage(imageId) {
      this.$confirm('此操作将会删除该图片，是否继续', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // 删除
        console.log('删除图片成功')
        this.refreshImageList()
        this.serviceDeleteRow(imageId).then(response => {
          this.$message(response.data.message)
          this.refreshImageList()
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
.image-box{
    display: flex;
    flex-direction: row;
}
.image-item {
    width: 150px;
    height: 150px;
    margin-left:10px;
}
.image-content{
    width: 150px;
    height: 150px;
}
.add-image-button {
    display: flex;
    flex-direction: row;
    border:1px solid #ccc;
    align-items: center;
    justify-content: center;
    border:1px solid #ccc;
}
.el-icon-plus{
    font-size: 50px;
}
</style>
