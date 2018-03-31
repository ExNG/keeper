<template>
  <div class="window">

    <transition appear
                enter-active-class="animated fadeInDown"
    >
      <header class="toolbar toolbar-header animated fadeInDown"
              v-if="showOverlay"
      >
        <div class="toolbar-actions">
          <div class="btn-group">
            <button class="btn btn-default"
                    v-for="(tab, index) in tabs"
                    :key="index"
                    @click="setActiveTab(Number(index))"
                    :class="{ 'active': activeTab === index }"
            >
              <span class="icon icon-doc-text-inv"></span>
              <small class="pull-left"
                     v-if="showTooltip && (Number(index) + 1 <= 10)"
                     style="margin-right: 2.5px;"
              >
                {{ index + 1 }}
              </small>
              {{ tab.name }}

              <q-context-menu>
                <q-list link separator>
                  <q-item v-close-overlay @click.native="renameTab(Number(index))">
                    <q-item-main label="Rename" />
                  </q-item>

                  <q-item v-close-overlay @click.native="archiveTab(Number(index))">
                    <q-item-main label="Archive" />
                  </q-item>

                  <q-item v-close-overlay @click.native="removeTab(Number(index))">
                    <q-item-main label="Close" />
                  </q-item>
                </q-list>
              </q-context-menu>
            </button>
          </div>

          <button class="btn btn-primary"
                  v-show="tabs.length === 0"
                  @click="addTab({})"
          >
            <span class="icon icon-plus icon-text" style="color: white;"></span>
            New Tab
          </button>

          <button class="btn btn-default"
                  v-show="tabs.length >= 1"
                  @click="addTab({})"
          >
            <span class="icon icon-plus-circled"></span>
          </button>

          <button class="btn btn-default"
                  @click="save()"
          >
            <span class="icon icon-floppy icon-text"></span>
            Save
          </button>

          <div class="btn-group">
            <button class="btn btn-default"
                    @click="togglePane('sm')"
                    :class="{ 'active': panes.sm }"
            >
              <span class="icon icon-bookmark"></span>
            </button>
            <button class="btn btn-default"
                    @click="togglePane('left')"
                    :class="{ 'active': panes.left }"
            >
              <span class="icon icon-pencil"></span>
            </button>
            <button class="btn btn-default"
                    @click="togglePane('right')"
                    :class="{ 'active': panes.right }"
            >
              <span class="icon icon-eye"></span>
            </button>
          </div>

          <div class="pull-right">
            <button class="btn btn-default btn-dropdown">
              <span class="icon icon-menu icon-text"></span>
              <!-- Menu -->
              <q-popover>
                <q-list separator link>
                  <q-item>
                    Export
                    <export v-if="tabs[activeTab] && tabs.length >= 1"
                            :content="tabs[activeTab].content"
                            :name="tabs[activeTab].name"
                            :active-tab="activeTab"
                    ></export>
                  </q-item>

                  <q-item v-close-overlay @click.native="$router.push('/settings')">
                    Settings
                  </q-item>

                  <q-item>
                    Debug
                    <debug></debug>
                  </q-item>

                  <q-item v-close-overlay @click.native="shortcutsModal = true">
                    Shortcuts

                    <q-modal v-model="shortcutsModal">
                      <div class="padded-more">
                        <shortcuts></shortcuts>
                        <div style="text-align: right;">
                          <q-btn color="primary" @click="shortcutsModal = false">Close</q-btn>
                        </div>
                      </div>
                    </q-modal>
                  </q-item>

                  <q-item v-close-overlay @click.native="aboutModal = true">
                    About

                    <q-modal v-model="aboutModal">
                      <div class="padded-more">
                        <about :packageInfo="packageInfo"></about>
                        <div style="text-align: right;">
                          <q-btn color="primary" @click="aboutModal = false">Close</q-btn>
                        </div>
                      </div>
                    </q-modal>
                  </q-item>
                </q-list>
              </q-popover>
            </button>
          </div>

        </div>
      </header>
    </transition>

    <div class="window-content">
      <div class="pane-group" style="overflow: hidden;">
        <transition appear
                    enter-active-class="animated fadeInLeft"
        >
          <div class="pane-sm sidebar"
               v-if="panes.sm"
          >
            <nav class="nav-group">
              <h5 class="nav-group-title">
                <span class="icon icon-folder"></span>
                Archive
              </h5>
              <span class="nav-group-item cursor-pointer"
                    v-for="(note, index) in archived"
                    :key="index"
                    @click="restoreArchivedTab(index)"
              >
                <span class="icon icon-doc-text"></span>
                {{ note.name }}
              </span>
              <span class="nav-group-item" v-show="archived.length === 0">
                <span class="icon icon-info-circled"></span>
                No archived notes
              </span>

              <h5 class="nav-group-title">
                <span class="icon icon-cloud"></span>
                Cloud
              </h5>
              <cloud-list @addTab="addTab($event); save()"
                          :cloud="cloud"
                          :packageInfo="packageInfo"
              ></cloud-list>
            </nav>
          </div>
        </transition>

        <transition appear
                    enter-active-class="animated fadeInUp"
        >
          <div class="pane padded-more"
               v-if="panes.left"
          >
            <div v-for="(tab, index) in tabs"
                 :key="index"
            >
              <edit-input :content="tab.content"
                          @update="setTabContent(index, $event)"
                          v-if="tabs[activeTab] && activeTab === Number(index)"
                          class="edit-input"
              ></edit-input>
            </div>
          </div>
        </transition>

        <transition appear
                    enter-active-class="animated fadeInUp"
        >
          <div class="pane padded-more"
               v-if="panes.right"
          >
            <div v-for="(tab, index) in tabs"
                 :key="index"
            >
              <markdown-preview :content="tab.content"
                                :id="'preview-' + index"
                                v-if="tabs[activeTab] && activeTab === Number(index)"
              ></markdown-preview>
            </div>

          </div>
        </transition>

        <transition appear
                    enter-active-class="animated fadeIn"
        >
          <div class="pane padded-more"
               v-if="!panes.left && !panes.right"
          >
            <div class="absolute-center" style="text-align: center;">
              <h4 style="color: lightgrey">Literally nothing to see here :(</h4>
            </div>
          </div>
        </transition>
      </div>
    </div>

    <transition appear
                enter-active-class="animated fadeInUp"
    >
      <footer class="toolbar toolbar-footer animated fadeInUp"
              v-if="showOverlay"
      >
        <div class="toolbar-actions">
          <div class="pull-right">
            <button class="btn btn-default pull-right"
                    @click="applyMarkdownStyle()"
            >
              <span class="icon icon-feather"></span>
            </button>
          </div>
        </div>
      </footer>
    </transition>
  </div>
</template>

<script>
import About from '../pages/About'
import Debug from '../pages/Debug'
import CloudList from '../pages/CloudList'
import EditInput from '../pages/EditInput'
import Export from '../pages/Export'
import ExportDialog from '../services/Export'
import Markdown from '../services/Markdown'
import MarkdownPreview from '../pages/MarkdownPreview'
import Mousetrap from 'mousetrap'
import Notification from '../services/Notification'
import Shortcuts from '../pages/Shortcuts'
import StartupHandler from '../services/StartupHandler'
import Storage from '../services/Storage'

export default {
  components: {
    About,
    Debug,
    CloudList,
    EditInput,
    Export,
    Shortcuts,
    MarkdownPreview
  },

  data () {
    return {
      packageInfo: require('../../package.json'),

      shortcutsModal: false,
      aboutModal: false,

      showTooltip: false,

      activeTab: 0,
      showOverlay: true,
      panes: {
        sm: false,
        left: true,
        right: true
      },

      settings: {},

      tabs: [],
      archived: [],
      cloud: {},

      newTab: {
        name: 'New Tab',
        content: ''
      }
    }
  },

  methods: {
    setActiveTab (tabIndex) {
      tabIndex = Number(tabIndex)
      if (!this.tabs[tabIndex]) {
        return
      }

      this.activeTab = tabIndex
    },

    addTab ({name = null, content = null}) {
      let data = JSON.parse(JSON.stringify(this.newTab))
      data.name = name ? String(name) : 'Unamed Tab'
      data.content = content ? String(content) : ''
      this.setActiveTab(Number(this.tabs.push(data)) - 1)

      this.save()
    },

    removeTab (index) {
      this.activeTab = Number(index - 1)

      if (this.activeTab <= 0) {
        this.activeTab = null
      }

      this.tabs.splice(Number(index), 1)

      this.save()
    },

    renameTab (index) {
      this.$q.dialog({
        title: 'Rename Tab',
        ok: true,
        cancel: true,

        prompt: {
          model: this.tabs[index].name,
          type: 'text'
        }
      })
        .then(data => {
          this.tabs[index].name = data
          this.save()
        })
    },

    archiveTab (index) {
      index = Number(index)
      this.archived.push(this.tabs[index])
      this.activeTab = index - 1
      this.tabs.splice(index, 1)

      this.save()
    },

    restoreArchivedTab (index) {
      index = Number(index)
      this.tabs.push(this.archived[index])
      this.activeTab = this.tabs.length - 1
      this.archived.splice(index, 1)

      this.save()
    },

    togglePane (paneName) {
      let settings = Storage.load('settings')

      this.panes[paneName] = !this.panes[paneName]
      settings.panes = this.panes

      Storage.save('settings', settings)
    },

    exportDoc () {
      ExportDialog.exportDialog(this.tabs[this.activeTab].content, this.tabs[this.activeTab].name, this.activeTab)
    },

    save () {
      Storage.save('archived', this.archived)
      Storage.save('tabs', this.tabs)
    },

    setTabContent (tabIndex, content) {
      this.tabs[tabIndex].content = content
    },

    applyMarkdownStyle () {
      let content = this.tabs[this.activeTab].content

      if (this.settings.replace) {
        let strings = this.settings.replaceList

        for (let string in strings) {
          while (content.indexOf(string) !== -1) {
            content = content.replace(string, strings[string])
          }
        }
      }

      content = Markdown.applyStyle(content)

      this.tabs[this.activeTab].content = String(content)

      this.$emit('content-updated')
    }
  },

  created () {
    StartupHandler()

    this.tabs = Storage.load('tabs')

    this.archived = Storage.load('archived')

    this.settings = Storage.load('settings')
    this.panes = this.settings.panes

    this.cloud = Storage.load('cloud')

    Mousetrap.bind('ctrl+s', (e) => {
      this.save()
      Notification({title: 'Saved', type: 'info'})
    })
    Mousetrap.bind('ctrl+p', (e) => {
      this.applyMarkdownStyle()
      Notification({title: 'Beautified', type: 'info'})
    })
    Mousetrap.bind('ctrl+n', (e) => { this.addTab({}) })
    Mousetrap.bind('ctrl+w', (e) => { this.archiveTab(Number(this.activeTab)) })
    Mousetrap.bind('ctrl+q', (e) => { require('electron').remote.app.quit() })
    Mousetrap.bind('option+down', (e) => { this.showOverlay = !this.showOverlay })
    Mousetrap.bind('option+up', (e) => { this.togglePane('left') })
    Mousetrap.bind('option+left', (e) => { this.togglePane('sm') })
    Mousetrap.bind('option+right', (e) => { this.togglePane('right') })
    Mousetrap.bind('option+s', (e) => { this.$router.push('/settings') })

    Mousetrap.bind('option', (e) => { this.showTooltip = true }, 'keydown')
    Mousetrap.bind('option', (e) => { this.showTooltip = false }, 'keyup')

    Mousetrap.bind('option+1', (e) => { this.setActiveTab(0) })
    Mousetrap.bind('option+2', (e) => { this.setActiveTab(1) })
    Mousetrap.bind('option+3', (e) => { this.setActiveTab(2) })
    Mousetrap.bind('option+4', (e) => { this.setActiveTab(3) })
    Mousetrap.bind('option+5', (e) => { this.setActiveTab(4) })
    Mousetrap.bind('option+6', (e) => { this.setActiveTab(5) })
    Mousetrap.bind('option+7', (e) => { this.setActiveTab(6) })
    Mousetrap.bind('option+8', (e) => { this.setActiveTab(7) })
    Mousetrap.bind('option+9', (e) => { this.setActiveTab(8) })
    Mousetrap.bind('option+0', (e) => { this.setActiveTab(9) })
  }
}
</script>

<style>
</style>