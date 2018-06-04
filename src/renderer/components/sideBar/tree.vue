<template>
  <div class="tree-view">
    <div class="title">
      <span>Explorer</span>
      <a
        href="javascript:;"
        :class="{'active': active === 'tree'}"
      >
        <svg class="icon" aria-hidden="true" @click="active = 'tree'">
          <use xlink:href="#icon-tree"></use>
        </svg>       
      </a>
      <a
        href="javascript:;"
        :class="{'active': active === 'list'}"
      >
        <svg class="icon" aria-hidden="true" @click="active = 'list'">
          <use xlink:href="#icon-list"></use>
        </svg>        
      </a>
    </div>
    <!-- opened files -->
    <div class="opened-files">
      <div class="title">
        <svg class="icon" aria-hidden="true">
          <use xlink:href="#icon-arrow"></use>
        </svg>
        <span>Opened files</span>
        <a href="javascript:;" @click.stop="saveAll(false)">
          <svg class="icon" aria-hidden="true">
            <use xlink:href="#icon-save-all"></use>
          </svg>
        </a>
        <a href="javascript:;" @click.stop="saveAll(true)">
          <svg class="icon" aria-hidden="true">
            <use xlink:href="#icon-close-all"></use>
          </svg>
        </a>
      </div>
      <opened-file
        v-for="(tab, index) of tabs"
        :key="index"
        :file="tab"
      ></opened-file>
    </div>
    <!-- project tree view -->
    <div 
      class="project-tree" v-if="projectTree"
    >
      <div class="title">
        <svg class="icon" aria-hidden="true">
          <use xlink:href="#icon-arrow"></use>
        </svg>
        <span>{{ projectTree.name }}</span>
      </div>
      <div class="tree-wrapper" v-show="active === 'tree'">
        <folder
          v-for="(folder, index) of projectTree.folders" :key="index + 'folder'"
          :folder="folder"
          :depth="depth"
        ></folder>
        <input
          type="text" class="new-input" v-show="createCache.dirname === projectTree.pathname"
          :style="{'margin-left': `${depth * 5 + 15}px` }"
          ref="input"
          v-model="createName"
          @keydown.enter="handleInputEnter"
        >
        <file
          v-for="(file, index) of projectTree.files" :key="index + 'file'"
          :file="file"
          :depth="depth"
        ></file>
      </div>
      <div v-show="active === 'list'">
        <list-file
          v-for="(file, index) of fileList"
          :key="index"
          :file="file"
        ></list-file>
      </div>
    </div>
    <div v-else class="open-project">
      <a href="javascript:;" @click="openProject">Open Project</a>
    </div>
  </div>
</template>

<script>
  import Folder from './folder.vue'
  import File from './file.vue'
  import ListFile from './listFile.vue'
  import OpenedFile from './openedFile.vue'
  import { mapState } from 'vuex'
  import bus from '../../bus'
  import { createFileOrDirectoryMixins } from '../../mixins'

  export default {
    mixins: [createFileOrDirectoryMixins],
    data () {
      this.depth = 0
      return {
        active: 'list', // tree or list
        showNewInput: false,
        createName: ''
      }
    },
    props: {
      projectTree: {
        validator: function (value) {
          return typeof value === 'object'
        },
        required: true
      },
      fileList: Array,
      openedFiles: Array,
      tabs: Array
    },
    components: {
      Folder,
      File,
      ListFile,
      OpenedFile
    },
    computed: {
      ...mapState({
        'createCache': state => state.project.createCache
      })
    },
    created () {
      this.$nextTick(() => {
        bus.$on('SIDEBAR::show-new-input', this.handleInputFocus)
        // hide rename or create input if needed
        document.addEventListener('click', event => {
          const target = event.target
          if (target.tagName !== 'INPUT') {
            this.$store.commit('CREATE_PATH', {})
            this.$store.commit('SET_RENAME_CACHE', null)
          }
        })
        document.addEventListener('contextmenu', event => {
          const target = event.target
          if (target.tagName !== 'INPUT') {
            this.$store.commit('CREATE_PATH', {})
            this.$store.commit('SET_RENAME_CACHE', null)
          }
        })
        document.addEventListener('keydown', event => {
          if (event.key === 'Escape') {
            this.$store.commit('CREATE_PATH', {})
            this.$store.commit('SET_RENAME_CACHE', null)
          }
        })
      })
    },
    methods: {
      titleIconClick (active) {
        //
      },
      openProject () {
        this.$store.dispatch('ASK_FOR_OPEN_PROJECT')
      },
      saveAll (isClose) {
        this.$store.dispatch('ASK_FOR_SAVE_ALL', isClose)
      }
    }
  }
</script>

<style scoped>
  .tree-view {
    font-size: 13px;
    color: var(--regularColor);
    display: flex;
    flex-direction: column;
    height: 100%;
  }
  .tree-view > .title {
    height: 35px;
    line-height: 35px;
    padding: 0 15px;
    display: flex;
    & > span {
      flex: 1;
      user-select: none;
    }
    & > a {
      pointer-events: auto;
      cursor: pointer;
      margin-left: 8px;
      color: var(--primaryColor);
    }
    & > a:hover {
      color: var(--brandColor);
    }
    & > a.active {
      color: var(--activeColor);
    }
  }

  .opened-files,
  .project-tree {
    & > .title {
      height: 25px;
      background: var(--lightBarColor);
      line-height: 25px;
    }
  }

  .opened-files .title {
    padding-right: 15px;
    display: flex;
    align-items: center;
    & > span {
      flex: 1;
    }
    & > a {
      display: none;
      text-decoration: none;
      color: var(--primaryColor);
      margin-left: 8px;
    }
  }
  .opened-files div.title:hover > a {
    display: block;
    &:hover {
      color: var(--brandColor);
    }
  }

  .project-tree {
    display: flex;
    flex-direction: column;
    & > .tree-wrapper {
      overflow: auto;
      flex: 1;
      &::-webkit-scrollbar:vertical {
        width: 5px;
      }
    }
    flex: 1;
  }
  .open-project {
    flex: 1;
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin-top: -100px;
    & > a {
      text-decoration: none;
      color: var(--activeColor);
    }
  }
  .new-input {
    outline: none;
    height: 18px;
    margin: 5px 0;
  }
</style>