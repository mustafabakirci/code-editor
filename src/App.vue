<template>
  <div id="app">
    <div class="menu">
      <select v-model="historySelect">
        <option value="0">History</option>
        <option :value="item.name" v-for="item in historyList" :key="item.name">{{item.name}}</option>
      </select>
      <button @click="openSavePopup">Save</button>
    </div>

    <iframe id="code" ref="code" :class="minimize ? 'min': ''"></iframe>
    <div class="editor-actions">
      <button @click="clearEditor">Clear</button>
      <button @click="minimize = !minimize">{{minimize ? 'Maximize' : 'Minimize'}}</button>
      <button @click="previewContent">Run</button>
    </div>
    <div class="editor-wrapper" :class="minimize ? 'min': ''">
      <CodeEditor :languages="[['html', 'HTML']]" width="auto" v-model="editor.html" :wrap_code="true" height="100%"></CodeEditor>
      <CodeEditor :languages="[['javascript', 'JS']]" width="auto" v-model="editor.js" :wrap_code="true" height="100%"></CodeEditor>
      <CodeEditor :languages="[['css', 'CSS']]" width="auto" v-model="editor.css" :wrap_code="true" height="100%"></CodeEditor>
    </div>
    <modal name="saveModal" height="auto">
      <div class="dialog-content"><input v-model="historyItemName" type="text" placeholder="Content Name"></input></div>
      <div class="vue-dialog-buttons">
        <button @click="save">Save</button>
      </div>
    </modal>
  </div>
</template>

<script>
import CodeEditor from 'simple-code-editor';

export default {
  name: 'App',
  data() {
    return {
      editor: {
        html: '',
        js: '',
        css: '',
      },
      historyList : [],
      historyItemName: '',
      minimize : false,
      historySelect : 0
    }
  },
  components: {
    CodeEditor
  },
  methods: {
    previewContent() {
      
      if (!this.validatehtml()) {
        alert('Hatalı bir html kodu girdiniz.')
        return;
      }
      let code = (this.$refs.code).contentWindow.document;
      code.open();
      code.writeln(``)
      code.writeln(`${this.editor.html}<style>${this.editor.css}</style><\script>${this.editor.js}<\/script>`)
      code.close();
    },
    clearEditor() {
      this.editor = {
        html: '',
        js: '',
        css: '',
      }
      this.previewContent();
    },
    openSavePopup() {
      this.$modal.show('saveModal');
    },
    save() {
      let foundIndex = this.historyList.findIndex((x) => x.name == this.historyItemName)
      if (foundIndex == -1) {
            this.historyList.push({...this.editor, name: this.historyItemName})
        } else {
            if (confirm("Existing content with the same name will be updated.")) {
              this.historyList[foundIndex] = {...this.editor, name: this.historyItemName};
            }
        }
      localStorage.setItem('historyList', JSON.stringify(this.historyList))
      this.$modal.hide('saveModal');
    },
    refreshProtect() {
      sessionStorage.setItem('draftCode', JSON.stringify(this.editor));
    },
    validatehtml() {
      var doc = document.createElement('div');
      doc.innerHTML = this.editor.html;
      return ( doc.innerHTML === this.editor.html );
    }
  },
  created() {
    let draftCode = sessionStorage.getItem('draftCode');
    if (draftCode) {
      this.editor = JSON.parse(draftCode)
    }
    let historyList = localStorage.getItem('historyList');
    if (historyList) {
       this.historyList = JSON.parse(historyList)
    }
  },
  mounted() {
    window.addEventListener("keydown", function(e) {
      if (e.ctrlKey && e.key === 's') {
        e.preventDefault();
        this.previewContent();
      }
    }.bind(this));
    window.addEventListener('beforeunload', function(event) {
      this.refreshProtect()
    }.bind(this));
  },
  watch: {
    historySelect: {
      handler(newQuestion) {
        if (newQuestion != 0) {
          let findedItem = this.historyList.find(item => item.name == newQuestion)
          // delete findedItem["name"];
          // console.log(findedItem)
          this.editor = findedItem
          this.previewContent()
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
#app {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: stretch;
  min-height: 100vh;
}
  .menu {
    background: $dark-grey;
    padding: 20px;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 10px;
    button {
      background: #8746BF;
      height: 25px;
      padding: 0 15px;
      color: $white;
    }
    select {
      background: $white;
      height: 25px;
      padding: 0 15px;
      color: $black;
      line-height: 25px;
      appearance: auto;
    }
  }
  .editor-actions {
    margin-top: auto;
    padding: 0 20px 20px;
    display: flex;
    gap: 15px;
    align-items: center;
    justify-content: flex-end;
    button {
      background: #F8F5DF;
      height: 45px;
      padding: 0 15px;

    }
  }
  .editor-wrapper {
    display: flex;
    gap: 20px;
    padding: 0 20px 30px;
    height: 40vh;
    transition: all ease-in-out .8s;
    .code_editor {
      flex: 1;
    }
    &.min {
      height: 64px;
    }
  }
  #code {
    height: 40vh;
    border: 1px solid $black;
    margin: 20px;
    transition: all ease-in-out .8s;
    &.min {
      height: 70vh;
    }
  }
  .dialog-content {
    padding: 25px;
    input {
      border: 1px solid $black;
      width: 100%;
      height: 45px;
      padding: 0 25px;
    }
  }
  .vue-dialog-buttons {
    padding: 25px;
    button {
      margin-left: auto;
      margin-right: auto;
      background: #8746BF;
      height: 45px;
      padding: 0 15px;
      color: $white;
    }
  }
</style>
