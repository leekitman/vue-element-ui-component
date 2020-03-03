<!--
版本：1.0.1
更新2020-02-03：增加字体大小功能

props说明：
序号	props属性名	        类型	      作用描述	                                            默认值	示例
1	    reviewContent	     String	     单向绑定content，主要用于指定数据回显属性	             ""     :review-content="form.content"

方法说明：
1、getLjUeditorContent：获取当前编辑器的内容，主要用于表单提交前调用，调用方法示例：
```
method: {
  beforeSubmitFun() {
    this.form.content = this.$refs.ljUeditor.getLjUeditorContent()
  }
}
```
-->
<template>
  <div class="ljUeditorDiv">
    <VueUEditor :ueditor-config="config" @ready="editorReady" :ueditor-path="editorPath"></VueUEditor>
  </div>
</template>
<script>
import VueUEditor from 'vue-ueditor'
export default {
  name: 'LjUeditor',
  components: {
    VueUEditor
  },
  props: {
    reviewContent: { type: String, default: '' },
    editorConfig: { type: Object, default: () => { return {} } }// 暂无用
  },
  data() {
    return {
      editorPath: '',
      editor: null,
      content: this.reviewContent,
      config: {
        initialFrameHeight: 300,
        toolbars: [
          [
            'fullscreen',
            'source',
            '|',
            'undo',
            'redo',
            '|',
            'bold',
            'italic',
            'underline',
            '|',
            'fontsize',
            'forecolor',
            'backcolor',
            '|',
            'justifyleft',
            'justifycenter',
            'justifyright',
            'justifyjustify',
            '|',
            'imagenone',
            'imageleft',
            'imageright',
            'imagecenter',
            '|',
            'simpleupload'
          ]
        ]
      }
    }
  },
  watch: {
    reviewContent: function(val, oldVal) {
      console.log('ueditor content import', val)
      // 这里做判断很关键：不做判断会导致出现该侦听器在`methods.editorReady()`方法之前执行，此时`this.editor`是undefned的，此时控制台报错并回显失败。
      if (this.editor) {
        this.editor.setContent(val)
      }
    }
  },
  created() {
    this.editorPath = process.env.EDITOR_BASE_PATH + '/static/ueditor1_4_3_3/'
    window.UEDITOR_HOME_URL = this.editorPath
    window.UEDITOR_SERVER_URL = process.env.EDITOR_SERVER_URL
  },
  methods: {
    editorReady(editorInstance) {
      console.log('ueditor is ready')
      editorInstance.addListener('contentChange', () => {
        console.log('内容变化', this.content)
        this.content = editorInstance.getContent()
      })
      this.editor = editorInstance
      this.editor.setContent(this.reviewContent)
    },
    getLjUeditorContent() {
      return this.editor.getContent()
    }
  }
}
</script>
<style lang="scss">
.ljUeditorDiv{
  min-height: 300px
}
</style>
