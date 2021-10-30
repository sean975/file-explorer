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
