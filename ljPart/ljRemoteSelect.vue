<!-- 远程搜索下拉框 -->
<!--
version:1.0.1
2020-07-09：兼容“值”和“对象”两种绑定模式
 -->
<template>
  <div>
    <el-select
      :style="selectWidth"
      v-model="miValue"
      @change="onChange"
      filterable
      remote
      default-first-option
      :placeholder="placeholder"
      :remote-method="remoteMethod"
      :loading="loading">

      <el-option
        v-for="item in options"
        :key="item[keyField]"
        :label="item[showField]"
        :value="item[keyField]">
      </el-option>

    </el-select>
  </div>
</template>

<script>
export default {
  name: 'LjRemoteSelect',
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    returnObj: { type: Boolean, default: false },
    value: { type: String / Object, default: '' / {}},
    showField: { type: String, default: 'text' },
    keyField: { type: String, default: 'id' },
    placeholder: { type: String, default: '请输入关键词' },
    disabled: { type: Boolean, default: false },
    serviceList: { type: Function, default: (keyword) => { console.error('远程搜索下拉框缺少远程api', keyword) } },
    width: { type: String, default: '' }
  },
  data() {
    return {
      data: this.value,
      loading: false,
      otherValue: this.value,
      options: []
    }
  },
  computed: {
    selectWidth() {
      if (this.width) {
        return 'width:' + this.width + ';'
      } else {
        return ''
      }
    },
    miValue: {
      get: function() {
        if (this.returnObj) {
          return this.value[this.keyField]
        } else {
          return this.value
        }
      },
      set: function(newValue) {
        this.data = newValue
      }
    }
  },
  methods: {
    remoteMethod(keyword) {
      if (keyword !== '') {
        this.loading = true
        this.serviceList(keyword).then(res => {
          this.options = res.data.data
          this.loading = false
        })
      } else {
        this.options = []
        this.loading = false
      }
    },
    onChange(value) {
      if (this.returnObj) {
        var index
        this.options.forEach((user, i) => {
          if (value === user.id) {
            index = i
          }
        })
        this.$emit('change', this.options[index])
      } else {
        this.$emit('change', value)
      }
    }
  }
}
</script>

<style>

</style>

