<template>
  <div class="announcement view">
    <Panel title="题目标签管理">
      <draggable v-model="tags" @end="onDragEnd">
      <el-tag v-for="tag in tags"
              :key="tag.name"
              type="ghost"
              shape="circle"
              closable
              @close="handleClose(tag)"
              class="tag-btn">{{tag.name}}
      </el-tag>
      </draggable>
      <el-input
        class="input-new-tag"
        v-if="inputVisible"
        v-model="inputValue"
        ref="saveTagInput"
        size="small"
        @keyup.enter.native="handleInputConfirm"
        @blur="handleInputConfirm"
      >
      </el-input>
      <el-button v-else class="button-new-tag" size="small" @click="showInput">+ New Tag</el-button>

    </Panel>
  </div>
</template>

<script>
  import Simditor from '../../components/Simditor.vue'
  import api from '../../api.js'
  import {mapGetters} from 'vuex'
  import draggable from 'vuedraggable'

  export default {
    name: 'Tags',
    components: {
      Simditor, draggable
    },
    computed: {
      ...mapGetters(['user', 'isSuperAdmin', 'isSecondary'])
    },
    data () {
      return {
        tags: [],
        // 是否显示loading
        loading: true,
        inputVisible: false,
        inputValue: ''
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.getTagsList()
      },
      getTagsList () {
        this.loading = true
        api.getTagsList().then(res => {
          this.loading = false
          this.tags = res.data.data
        }, res => {
          this.loading = false
        })
      },
      showInput () {
        this.inputVisible = true
        this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus()
        })
      },
      handleInputConfirm () {
        console.log(Math.max(...this.tags.map(o => o.order)) + 1)
        let inputValue = this.inputValue
        if (inputValue) {
          api.createTag({
            name: inputValue,
            order: Math.max(...this.tags.map(o => o.order)) + 1
          }).then(res => {
            this.init()
          })
        }
        this.inputVisible = false
        this.inputValue = ''
      },
      deleteTag (tagId) {
        this.$confirm('Are you sure you want to delete this tag?', 'Warning', {
          confirmButtonText: 'Delete',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          // then 为确定
          this.loading = true
          api.deleteTag(tagId).then(res => {
            this.loading = true
            this.init()
          })
        }).catch(() => {
          // catch 为取消
          this.loading = false
        })
      },
      handleClose (tag) {
        this.deleteTag(tag.id)
      },
      onDragEnd () {
        let cnt = this.tags.length
        for (let tag of this.tags) {
          tag.order = cnt
          cnt--
        }
        api.updateTag(this.tags).then(res => {
          this.init()
        })
      }
    },
    watch: {
      $route () {
        this.init()
      }
    }
  }
</script>

<style lang="less" scoped>
  .title-input {
    margin-bottom: 20px;
  }

  .tag-btn {
    margin-right: 5px;
    margin-bottom: 10px;
  }

  .visible-box {
    margin-top: 10px;
    width: 205px;
    float: left;
  }
</style>
