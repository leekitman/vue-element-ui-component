<template>
  <div>
    <div class="file-input-box">
      <el-button type="primary">点击选择文件<i class="el-icon-upload el-icon--right"></i></el-button>
      <input class="file-input" type="file" @change="onFileChange" :multiple="multipleValue"/>
      <i v-if="hasFile" class="el-icon-delete" @click="clearFiles"></i>
    </div>
    <!-- 预览区 -->
    <span class="error-message" v-if="hasError">上传图片不能超过{{ maxCount }}张</span>
    <div class="preview-box">
      <div v-for="(file, index) in fileArray" :key="index">
        <el-card class="box-card">
          <div style="margin: 5px;" class="preview-box-item">
            <img :src="getFileUrl(file)" height="100" v-if="isImageFile(file)"/>
            <div v-if="file">已选择：{{ file.name }}</div>
          </div>
        </el-card>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'FileInput',
  model: { // 定义组件v-model属性，并把v-model属性绑定到属性fileData
    prop: 'fileData',
    event: 'change' // https://cn.vuejs.org/v2/guide/components-custom-events.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E7%9A%84-v-model
  },
  props: {
    fileData: {
      type: FileList,
      required: false,
      default: null
    },
    multiple: { type: Boolean, default: false },
    maxCount: { type: Number, default: 5 }
  },
  data() {
    return {
      fileArray: this.fileData,
      hasError: false
    }
  },
  computed: {
    multipleValue: {
      cache: false,
      get: function() {
        if (this.multiple) {
          return 'multiple'
        } else {
          ''
        }
      }
    },
    hasFile: {
      cache: false,
      get: function() {
        if (this.fileArray && this.fileArray.length > 0) {
          return true
        } else {
          false
        }
      }
    }
  },
  methods: {
    clearFiles() {
      this.fileArray = null
      this.$emit('change', this.fileArray)
    },
    onFileChange(event) {
      console.log(event.target.files, '查看选择的图片')
      this.fileArray = event.target.files
      if (this.fileArray.length > this.maxCount) {
        console.error('上传图片不能超过' + this.maxCount + '张')
        this.hasError = true
        this.fileArray = null
      } else {
        this.hasError = false
      }
      this.$emit('change', this.fileArray)
    },
    getFileUrl(file) {
      if (!file) {
        return ''
      }
      return window.URL.createObjectURL(file)
    },
    isImageFile(file) {
      console.log('检查文件类型', file)
      if (file) {
        return file.type === 'image/jpeg' || file.type === 'image/png'
      } else {
        return false
      }
    },
    getFileExt(file) {
      if (!file) {
        return ''
      }
      const path = file.name
      const dotIndex = path.lastIndexOf('.')
      const ext = path.substr(dotIndex + 1).toLowerCase()
      return ext
    }
  }
}
</script>
<style lang="scss" scoped>
.file-input-box {
  position: relative;
}
.file-input-box input {
  position: absolute;
  top: 8px;
  left: 2px;
  opacity: 0;
  width: 140px
}
.preview-box-item {

}
.error-message {
  color: red
}
</style>
