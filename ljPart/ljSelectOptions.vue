<!--
版本：1.0.2
更新2020-02-27：支持远程数据懒加载树机制
-->
<template>
  <div>
    <template v-for="item in options">
      <el-option :key="item[valueKey]" :label="item[showField]" :value="valueObject?item:item[valueKey]" filterable :style="optionsStyle">
        <span v-if="level > 0" style="float: left;color: red">└</span>
        <i class="el-icon-caret-bottom" v-if="item.expand && ((local && item.children && item.children.length > 0) || !local)" @click.stop="nodeIconClickRight(item)"></i>
        <i class="el-icon-caret-right" v-else-if="!item.expand && ((local && item.children && item.children.length > 0) || !local)" @click.stop="nodeIconClickBottom(item)"></i>
        {{ item[showField] }}
      </el-option>
      <lj-select-options
        v-if="item.expand && ((local && item.children && item.children.length > 0) || !local)"
        :show-field="showField"
        :level="level + 1"
        :expand-all="expandAll"
        :value-key="valueKey"
        :value-object="valueObject"
        :local="local"
        :local-data="item.children"
        :load-parent-id="item.id"
        :load="load"
        :key="item[valueKey] + 'chil'" />
    </template>
  </div>
</template>
<script>
export default {
  name: 'LjSelectOptions',
  props: {
    // 通用配置
    showField: { type: String, default: 'text' },
    level: { type: Number, default: 0 },
    expandAll: { type: Boolean, default: false },
    valueKey: { type: String, default: 'id' },
    valueObject: { type: Boolean, default: false },
    // 本地数据配置
    local: { type: Boolean, default: true },
    localData: { type: Array, default: function() { return [] } },
    // 远程数据配置
    loadParentId: { type: String, default: '0' },
    load: { type: Function, default: function(node, resolve) {} }
  },
  data: function() {
    return {
      options: []
    }
  },
  computed: {
    optionsStyle() {
      return 'margin-left:' + this.level * 10 + 'px'
    }
  },
  created() {
    if (this.local) {
      this.options = this.localData.map(this.addExpandIntoNode)
      console.log('加载本地数据', this.localData)
    } else {
      this.load(this.loadParentId, (rootArray) => {
        var tempArray = rootArray.map(this.addExpandIntoNode)
        this.options = tempArray
      })
      console.log('加载远程数据', this.localData, this.options)
    }
  },
  methods: {
    nodeIconClickBottom(item) {
      console.log('节点图标点击事件-展开节点', item)
      item.expand = true
    },
    nodeIconClickRight(item) {
      console.log('节点图标点击事件-收缩节点', item)
      item.expand = false
    },
    addExpandIntoNode(node) {
      if (this.expandAll) {
        return { ...node, expand: true, isLeaf: false }
      } else {
        return { ...node, expand: false, isLeaf: false }
      }
    }
  }
}
</script>
