<template>
  <div>
    <textarea
      ref="mycode"
      v-model="value"
      class="codesql"
      :placeholder="$t('table.sql.enter')"
    />
  </div>
</template>
<script>
import 'codemirror/theme/idea.css'
import 'codemirror/lib/codemirror.css'
import 'codemirror/addon/hint/show-hint.css'
const CodeMirror = require('codemirror/lib/codemirror')
require('codemirror/addon/edit/matchbrackets')
require('codemirror/addon/selection/active-line')
require('codemirror/mode/sql/sql')
require('codemirror/addon/hint/show-hint')
require('codemirror/addon/hint/sql-hint')
require('codemirror/addon/display/placeholder')

export default {
  props: {
    value: {
      type: String,
      default: ''
    },
    sqlStyle: {
      type: String,
      default: 'default'
    },
    readOnly: {
      type: [Boolean, String],
      default: false
    }
  },
  data() {
    return {
      editor: null
    }
  },
  computed: {
    newVal() {
      if (this.editor) {
        return this.editor.getValue()
      } else {
        return null
      }
    }
  },
  watch: {
    newVal(newV, oldV) {
      if (this.editor) {
        this.$emit('changeTextarea', this.editor.getValue())
      }
    }
  },
  mounted() {
    const mime = 'text/x-mariadb'
    const theme = 'idea' // https://codemirror.net/demo/theme.html
    this.editor = CodeMirror.fromTextArea(this.$refs.mycode, {
      value: this.value,
      mode: mime,
      indentWithTabs: true,
      smartIndent: true,
      lineNumbers: true,
      matchBrackets: true,
      cursorHeight: 1,
      lineWrapping: true,
      readOnly: this.readOnly,
      theme: theme,
      // autofocus: true,
      extraKeys: { 'Ctrl': 'autocomplete' },
      hintOptions: {
        completeSingle: false
      }
    })
    this.editor.on('inputRead', () => {
      this.editor.showHint()
    })
  },
  methods: {
    setVal() {
      if (this.editor) {
        if (this.value === '') {
          this.editor.setValue('')
        } else {
          this.editor.setValue(this.value)
        }
      }
    }
  }
}
</script>
<style>
  .CodeMirror {
    color: black;
    direction: ltr;
    line-height: 24px;
    font-size: large;
    height: 250px;
  }
     .CodeMirror-hints{
       z-index: 9999 !important;
     }
  .codesql {
    overflow: hidden;
    border: 1px solid #eaeaea;
  }
</style>
