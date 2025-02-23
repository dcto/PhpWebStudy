<template>
  <div class="host-edit">
    <div class="nav">
      <div class="left" @click="doClose">
        <yb-icon :svg="import('@/svg/delete.svg?raw')" class="top-back-icon" />
        <span class="ml-15">Php Obfuscator Config</span>
      </div>
      <el-dropdown split-button type="primary" @click="doSave" @command="handleCommand">
        {{ $t('base.confirm') }}
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item command="import">{{ $t('base.import') }}</el-dropdown-item>
            <el-dropdown-item command="export">{{ $t('base.export') }}</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>

    <div class="main-wapper">
      <div ref="input" class="block" style="width: 100%; height: 100%"></div>
    </div>
  </div>
</template>

<script lang="ts">
  import { defineComponent, nextTick } from 'vue'
  import { readFileAsync, writeFileAsync } from '@shared/file'
  import { editor } from 'monaco-editor/esm/vs/editor/editor.api.js'
  import 'monaco-editor/esm/vs/editor/contrib/find/browser/findController.js'
  import 'monaco-editor/esm/vs/basic-languages/php/php.contribution.js'
  import { EditorConfigMake } from '@/util/Editor'
  import { MessageError, MessageSuccess } from '@/util/Element'

  const { join } = require('path')
  const { dialog } = require('@electron/remote')
  const { statSync, readFileSync } = require('fs')
  let config = ''
  export default defineComponent({
    components: {},
    props: {
      customConfig: {
        type: String,
        default: ''
      }
    },
    emits: ['doClose'],
    data() {
      return {
        config: ''
      }
    },
    computed: {},
    watch: {},
    created: function () {},
    mounted() {
      if (!config) {
        const file = join(global.Server.Static, 'tmpl/yakpro-po.default.cnf')
        config = readFileSync(file, 'utf-8')
      }
      this.config = this.customConfig || config
      nextTick().then(() => {
        this.initEditor()
      })
    },
    unmounted() {
      this.monacoInstance && this.monacoInstance.dispose()
      this.monacoInstance = null
    },
    methods: {
      handleCommand(command: 'import' | 'export') {
        switch (command) {
          case 'import':
            this.loadCustom()
            break
          case 'export':
            this.saveCustom()
            break
        }
      },
      loadCustom() {
        let opt = ['openFile', 'showHiddenFiles']
        dialog
          .showOpenDialog({
            properties: opt
          })
          .then(({ canceled, filePaths }: any) => {
            if (canceled || filePaths.length === 0) {
              return
            }
            const file = filePaths[0]
            const state = statSync(file)
            if (state.size > 5 * 1024 * 1024) {
              MessageError(this.$t('base.fileBigErr'))
              return
            }
            readFileAsync(file).then((conf) => {
              this.config = conf
              this.initEditor()
            })
          })
      },
      saveCustom() {
        let opt = ['showHiddenFiles', 'createDirectory', 'showOverwriteConfirmation']
        dialog
          .showSaveDialog({
            properties: opt,
            defaultPath: 'php-obfuscator.cnf',
            filters: [
              {
                extensions: ['cnf']
              }
            ]
          })
          .then(({ canceled, filePath }: any) => {
            if (canceled || !filePath) {
              return
            }
            const content = this.monacoInstance.getValue()
            writeFileAsync(filePath, content).then(() => {
              MessageSuccess(this.$t('base.success'))
            })
          })
      },
      doSave() {
        const content = this.monacoInstance.getValue().trim()
        this.$emit('doClose', content !== config ? content : undefined)
      },
      initEditor() {
        if (!this.monacoInstance) {
          const input: HTMLElement = this?.$refs?.input as HTMLElement
          if (!input || !input?.style) {
            return
          }
          const editorConfig = EditorConfigMake(this.config, false, 'off')
          editorConfig.language = 'php'
          this.monacoInstance = editor.create(input, editorConfig)
        } else {
          this.monacoInstance.setValue(this.config)
        }
      },
      doClose() {
        this.$emit('doClose')
      }
    }
  })
</script>
