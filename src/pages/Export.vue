<template>
  <q-popover>
    <q-list separator link>
      <q-item v-close-overlay
              @click.native="exportToTxt()"
      >
        Text
      </q-item>
      <q-item v-close-overlay
              @click.native="exportToHtml()"
      >
        Html
      </q-item>
      <q-item v-close-overlay>
        Cancel
      </q-item>
    </q-list>
  </q-popover>
</template>

<script>
import FileSaver from 'file-saver'
import Markdown from '../services/Markdown'

export default {
  props: ['content', 'name', 'activeTab'],

  data () {
    return {}
  },

  methods: {
    exportToTxt () {
      FileSaver.saveAs(
        new Blob(
          [ this.content ],
          {
            type: 'text/plain;charset=utf-8'
          }
        ),
        this.name + '.txt'
      )
    },

    exportToHtml () {
      FileSaver.saveAs(
        new Blob(
          [ Markdown.generateHtmlPage(this.content, this.name) ],
          {
            type: 'text/plain;charset=utf-8'
          }
        ),
        this.name + '.html'
      )
    }
  }
}
</script>

<style>
#debug-modal .btn {
  min-width: 80%;
}
</style>
