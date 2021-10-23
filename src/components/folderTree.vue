<template>
  <div>
    <q-separator />
    <q-expansion-item
      v-model="expanded"
      icon="folder_open"
      :label="isWin ? 'Drives' : 'File System'"
    >
      <q-separator />
      <q-tree
        ref="folders"
        :nodes="rootDir"
        label-key="label"
        node-key="nodeKey"
        accordion
        default-expand-all
        :selected.sync="selected"
        @lazy-load="lazyLoad"
      />
    </q-expansion-item>
    <q-separator />
  </div>
</template>

<script>
const path = require('path')

export default {
  name: 'FolderTree',

  props: {
    rootDir: {
      type: Array,
      required: true
    },
    folder: {
      type: String,
      required: false
    },
    lazyLoad: {
      type: Function,
      required: true
    }
  },

  data () {
    return {
      expanded: true,
      selected: null,
      isWin: process.platform === 'win32'
    }
  },

  mounted () {
    this.selected = this.folder

    this.$root.$on('expand-tree', this.expandTree)
  },

  beforeDestroy () {
    this.$root.$off('expand-tree', this.expandTree)
  },

  watch: {
    // this comes from the parent
    folder () {
      this.selected = this.folder
      this.expandTree(this.folder)
    },

    selected () {
      this.$emit('selected', this.selected)
    }
  },

  methods: {
    expandTree (absolutePath) {
      // get parts for the path
      const parts = absolutePath.split(path.sep)
      let path2 = ''
      let lastNodeKey

      // iterate through the path parts.
      // This code will get the node.
      for (let index = 0; index < parts.length; ++index) {
        if (parts[index].length === 0) {
          continue
        }
        if (index === 0) {
          path2 += parts[index] + path.sep
        }
        else {
          if (path2[path2.length - 1] !== path.sep) {
            path2 += path.sep
          }
          path2 += parts[index]
        }
        if (index > -1) {
          if (this.$refs.folders) {
            const key = this.$refs.folders.getNodeByKey(path2)
            // if we get key, then this folder has already been loaded
            if (key) {
              lastNodeKey = key
            }
            // handle folder not expanded
            if (!this.$refs.folders.isExpanded(lastNodeKey.nodeKey)) {
              this.$refs.folders.setExpanded(lastNodeKey.nodeKey, true)
              if (path2 === absolutePath) {
                this.selected = absolutePath
              }
              else {
                this.$nextTick(() => {
                  this.$root.$emit('expand-tree', absolutePath)
                })
              }
            }
          }
        }
      }
    }
  }
}
</script>
