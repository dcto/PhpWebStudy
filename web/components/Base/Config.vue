<template>
  <div class="module-config">
    <div ref="input" class="block"></div>
    <div class="tool">
      <el-button @click="openConfig">{{ $t('base.open') }}</el-button>
      <el-button @click="saveConfig">{{ $t('base.save') }}</el-button>
      <el-button @click="getDefault">{{ $t('base.loadDefault') }}</el-button>
      <el-button-group style="margin-left: 12px">
        <el-button @click="loadCustom">{{ $t('base.loadCustom') }}</el-button>
        <el-button @click="saveCustom">{{ $t('base.saveCustom') }}</el-button>
      </el-button-group>
    </div>
  </div>
</template>

<script lang="ts">
  import { editor, KeyCode, KeyMod } from 'monaco-editor/esm/vs/editor/editor.api.js'
  import 'monaco-editor/esm/vs/editor/contrib/find/browser/findController.js'
  import 'monaco-editor/esm/vs/basic-languages/ini/ini.contribution.js'
  import { defineComponent } from 'vue'
  import { EditorConfigMake } from '../../fn'

  export default defineComponent({
    name: 'MoApacheConfig',
    components: {},
    props: {
      config: {
        type: String,
        default: ''
      }
    },
    data() {
      return {
        typeFlag: 'apache',
        configpath: ''
      }
    },
    computed: {},
    watch: {},
    created: function () {},
    mounted() {
      this.initEditor()
    },
    unmounted() {
      this.monacoInstance && this.monacoInstance.dispose()
      this.monacoInstance = null
    },
    methods: {
      loadCustom() {},
      saveCustom() {},
      openConfig() {},
      saveConfig() {},
      getConfig() {},
      getDefault() {
        this.initEditor()
      },
      initEditor() {
        if (!this.monacoInstance) {
          const input: HTMLElement = this?.$refs?.input as HTMLElement
          if (!input || !input?.style) {
            return
          }
          this.monacoInstance = editor.create(input, EditorConfigMake(this.config, false, 'off'))
          this.monacoInstance.addAction({
            id: 'save',
            label: 'save',
            keybindings: [KeyMod.CtrlCmd | KeyCode.KeyS],
            run: () => {
              this.saveConfig()
            }
          })
        } else {
          this.monacoInstance.setValue(this.config)
          this.monacoInstance.updateOptions({
            readOnly: false
          })
        }
      }
    }
  })
</script>
