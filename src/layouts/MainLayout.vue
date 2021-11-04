<template>
  <q-layout view="hHh LpR lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <breadcrumbs
          :absolutePath="selectedFolder"
          @selected="onSelectedFolder"
        />

        <q-btn
          flat
          dense
          round
          :icon="listType === 'grid' ? 'format_list_bulleted' : 'border_all'"
          aria-label="toggle between grid and list modes"
          @click="toggleListType"
        />

      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      side="left"
      show-if-above
      behavior="desktop"
      bordered
      content-class="bg-grey-1"
    >
      <shortcuts
        @selected="onShortcutSelected"
      />
      <folder-tree
        :rootDir="rootDir"
        :folder.sync="selectedFolder"
        :lazyLoad="onLazyLoad"
        @selected="onSelectedFolder"
      />
    </q-drawer>

    <q-page-container>
      <!-- <router-view /> -->

      <contents
        :contents="contents"
        :listType="listType"
        @click="onClicked"
        @dblClick="onDblClicked"
      />

    </q-page-container>
  </q-layout>
</template>

<script>
const { ipcRenderer } = require('electron')
const { app } = require('@electron/remote')
const mime = require('mime-types')
const chokidar = require('chokidar') // file watcher

const fs = require('fs')
const path = require('path')

export default {
  name: 'MainLayout',
  components: {
    Breadcrumbs: () => import('../components/breadcrumbs'),
    Shortcuts: () => import('../components/shortcuts'),
    FolderTree: () => import('../components/folderTree'),
    Contents: () => import('../pages/contents')
  },

  data () {
    return {
      leftDrawerOpen: false,
      toolbarLinks: [], // toolbar pathway (links to each folder in path)
      drive: process.platform === 'win32' ? 'C:' : '',
      drives: [],
      listType: 'grid', // default ['grid', 'list']
      selectedFolder: null, // the selected node (label)
      rootDir: [], // tree view
      contents: [], // children of a node
      watcher: null
    }
  },

  methods: {
    onSelectedFolder (absolutePath) {
      this.setSelectedFolder(absolutePath)
    },

    onShortcutSelected (type) {
      const absolutePath = app.getPath(type)
      this.setSelectedFolder(absolutePath)
    },

    setSelectedFolder (folder) {
      this.selectedFolder = folder
      // handle windows drive
      if (process.platform === 'win32') {
        if (this.selectedFolder.charAt(this.selectedFolder.length - 1) === ':') {
          this.selectedFolder += path.sep
        }
      }
    },

    onClicked (node) {
      // do nothing
    },

    onDblClicked (node) {
      // open a folder
      if (node.data.isDir) {
        this.setSelectedFolder(node.nodeKey)
      }
      else {
        this.onFileSelected(node)
      }
    },

    onFileSelected (node) {
      console.log('selected:', node)
    },

    toggleListType () {
      if (this.listType === 'grid') {
        this.listType = 'list'
      }
      else {
        this.listType = 'grid'
      }
    },

    clearAllContentItems () {
      this.contents.splice(0, this.contents.length)
    },

    addContentItem (item) {
      this.contents.push(item)
    },

    createNode (fileInfo) {
      let nodeKey = fileInfo.rootDir
      if (nodeKey.charAt(nodeKey.length - 1) !== path.sep) {
        nodeKey += path.sep
      }
      if (fileInfo.fileName === path.sep) {
        fileInfo.fileName = nodeKey
      }
      else {
        nodeKey += fileInfo.fileName
      }
      // get file mime type
      const mimeType = mime.lookup(nodeKey)
      // create object
      return {
        label: fileInfo.fileName,
        nodeKey: nodeKey,
        expandable: fileInfo.isDir,
        tickable: true,
        lazy: true,
        children: [],
        data: {
          rootDir: fileInfo.rootDir,
          isDir: fileInfo.isDir,
          mimeType: mimeType,
          stat: fileInfo.stat
        }
      }
    },

    folderWatcherHandler (newFolder, oldFolder) {
      if (oldFolder && this.watcher) {
        this.watcher.close()
      }
      if (newFolder) {
        // let backend know
        ipcRenderer.send('folder', newFolder)

        this.watcher = chokidar.watch(newFolder, {
          depth: 0,
          ignorePermissionErrors: true
        })
        if (this.watcher) {
          this.watcher.on('ready', () => {
            // watch for additions
            this.watcher.on('raw', (event, path, details) => {
              this.$root.$emit('rescan-current-folder')
            })
          })
          this.watcher.on('error', (error) => {
            console.error(error)
          })
        }
      }
    }
  }

}
</script>

<style>
.breadcrumbs {
  width: calc(100% - 150px);
  height: 20px;
  margin-left: 5px;
  background-color: rgba(0, 0, 0, .3);
  border-radius: 4px;
}

.breadcrumb {
  cursor: pointer;
}

.breadcrumb:hover {
  text-decoration: underline;
}

.border {
  border-style: solid;
  border-radius: 2px;
  border-width: 1px;
  border-color: rgb(158, 158, 158);
  background-color: rgba(158, 158, 158, 0.1);
}

.q-layout-drawer {
  height: calc(100vh - 50px) !important;
}

.drawer-wrapper {
  position: relative;
  width: 100%;
  height: calc(100vh - 50px) !important;
}

.drawer-container {
  position: relative;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
}

</style>
