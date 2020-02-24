<template>
  <div>
    <el-tag
      :key="tag"
      v-for="tag in value"
      closable
      :disable-transitions="false"
      @close="handleClose(tag)">
      {{ tag }}
    </el-tag>
    <el-input
      class="input-new-tag"
      v-if="inputVisible"
      v-model="inputValue"
      ref="saveTagInput"
      size="small"
      @keyup.enter.native="handleInputConfirm"
      @blur="handleInputConfirm"
    >
    </el-input>
    <el-button v-else class="button-new-tag" size="small" @click="showInput">+ New Tag</el-button>
  </div>
</template>
<style>
  .el-tag + .el-tag {
    margin-left: 10px;
  }
  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }
  .input-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }
</style>
<script>
export default {
  name: 'LjTag',
  props: {
    value: { type: Array, default: function() { return [] } }
  },
  data: function() {
    return {
      dynamicTags: [],
      inputVisible: false,
      inputValue: ''
    }
  },
  methods: {
    onChange(value) {
      this.$emit('input', value)
    },
    handleClose(tag) {
      this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1)
    },
    handleInputConfirm() {
      const inputValue = this.inputValue
      if (inputValue) {
        this.dynamicTags.push(inputValue)
      }
      this.inputVisible = false
      this.inputValue = ''
      var newArray = []
      this.onChange(newArray.concat(this.value, this.dynamicTags))
      this.dynamicTags = []
    },
    showInput() {
      this.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    }
  }
}
</script>
