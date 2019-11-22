<template>
  <div>
    <template v-for="item in options">
      <el-option :key="item.id" :label="item[showField]" :value="item" filterable :style="optionsStyle">
        <span v-if="level > 0" style="float: left;color: red">└</span>
        {{ item[showField] }}
      </el-option>
      <LjObjectSelectOptions v-if="item.children && item.children.length > 0" :key="item.id + 'chil'" :options="item.children" :show-field="showField" :level="level + 1"/>
    </template>
  </div>
</template>
<script>
export default {
  name: 'LjObjectSelectOptions',
  props: {
    options: { type: Array, default: function() { [] } },
    showField: { type: String, default: 'text' },
    level: { type: Number, default: 0 }
  },
  data: function() {
    return {
    }
  },
  computed: {
    optionsStyle() {
      return 'margin-left:' + this.level * 10 + 'px'
    }
  }
}
</script>
