<template>
  <div class="announcement view">
    <Panel title="友情链接管理">
      <div class="list">
        <el-table
          v-loading="loading"
          element-loading-text="loading"
          ref="table"
          :data="friendshipLinksList"
          style="width: 100%">
          <el-table-column
            width="100"
            prop="id"
            label="ID">
          </el-table-column>
          <el-table-column
            prop="title"
            label="Title">
          </el-table-column>
          <el-table-column
            prop="create_time"
            label="CreateTime">
            <template slot-scope="scope">
              {{ scope.row.create_time | localtime }}
            </template>
          </el-table-column>
          <el-table-column
            prop="order"
            label="Order">
          </el-table-column>
          <el-table-column
            prop="created_by.username"
            label="Author">
          </el-table-column>
          <el-table-column
            v-if="isSuperAdmin"
            width="100"
            prop="visible"
            label="Visible">
            <template slot-scope="scope">
              <el-switch v-model="scope.row.visible"
                         active-text=""
                         inactive-text=""
                         @change="handleVisibleSwitch(scope.row)">
              </el-switch>
            </template>
          </el-table-column>
          <el-table-column
            v-if="isSuperAdmin"
            fixed="right"
            label="Option"
            width="200">
            <div slot-scope="scope">
              <icon-btn name="Edit" icon="edit" @click.native="openFriendshipLinksDialog(scope.row.id)"></icon-btn>
              <icon-btn name="Delete" icon="trash" @click.native="deleteFriendshipLinks(scope.row.id)"></icon-btn>
            </div>
          </el-table-column>
        </el-table>
        <div class="panel-options">
          <el-button type="primary" size="small" @click="openFriendshipLinksDialog(null)" icon="el-icon-plus">Create</el-button>
        </div>
      </div>
    </Panel>
    <!--对话框-->
    <el-dialog :title="friendshipLinksDialogTitle" :visible.sync="showEditFriendshipLinksDialog"
               @open="onOpenEditDialog" :close-on-click-modal="false">
      <el-form label-position="top">
        <el-form-item label="标题" required>
          <el-input
            v-model="friendshipLinks.title"
            placeholder="标题" class="title-input">
          </el-input>
        </el-form-item>
        <el-form-item label="链接" required>
          <el-input
            v-model="friendshipLinks.link"
            placeholder="链接" class="title-input">
          </el-input>
        </el-form-item>
        <el-form-item label="排序优先级（越高越靠前）" required>
          <el-input-number
            v-model="friendshipLinks.order"
            placeholder="排序优先级（越高越靠前）" class="title-input">
          </el-input-number>
        </el-form-item>
        <div class="visible-box">
          <span>是否可见</span>
          <el-switch
            v-model="friendshipLinks.visible"
            active-text=""
            inactive-text="">
          </el-switch>
        </div>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <cancel @click.native="showEditFriendshipLinksDialog = false"></cancel>
          <save type="primary" @click.native="submitFriendshipLinks"></save>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  import Simditor from '../../components/Simditor.vue'
  import api from '../../api.js'
  import {mapGetters} from 'vuex'

  export default {
    name: 'FriendshipLinks',
    components: {
      Simditor
    },
    computed: {
      ...mapGetters(['user', 'isSuperAdmin', 'isSecondary'])
    },
    data () {
      return {
        showEditFriendshipLinksDialog: false,
        friendshipLinksList: [],
        currentFriendshipLinksId: null,
        mode: 'create',
        friendshipLinks: {
          title: '',
          visible: true,
          link: '',
          order: 0
        },
        // 对话框标题
        friendshipLinksDialogTitle: 'Edit FriendshipLinks',
        // 是否显示loading
        loading: true,
        // 当前页码
        currentPage: 0
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.getFriendshipLinksList()
      },
      uploadImageSuccess (response, file, fileList) {
        this.friendshipLinks.link = response.file_path
      },
      getFriendshipLinksList () {
        this.loading = true
        api.getFriendshipLinksList().then(res => {
          this.loading = false
          this.friendshipLinksList = res.data.data
        }, res => {
          this.loading = false
        })
      },
      // 打开编辑对话框的回调
      onOpenEditDialog () {
        // todo 优化
        // 暂时解决 文本编辑器显示异常bug
        setTimeout(() => {
          if (document.createEvent) {
            let event = document.createEvent('HTMLEvents')
            event.initEvent('resize', true, true)
            window.dispatchEvent(event)
          } else if (document.createEventObject) {
            window.fireEvent('onresize')
          }
        }, 0)
      },
      submitFriendshipLinks (data = undefined) {
        if (!data.title) {
          data = {
            id: this.currentFriendshipLinksId,
            title: this.friendshipLinks.title,
            link: this.friendshipLinks.link,
            order: this.friendshipLinks.order,
            visible: this.friendshipLinks.visible
          }
        }
        let func = this.mode === 'edit' ? api.updateFriendshipLinks : api.createFriendshipLinks
        func(data).then(res => {
          this.showEditFriendshipLinksDialog = false
          this.init()
        })
      },
      deleteFriendshipLinks (friendshipLinksId) {
        this.$confirm('Are you sure you want to delete this FriendshipLinks?', 'Warning', {
          confirmButtonText: 'Delete',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          // then 为确定
          this.loading = true
          api.deleteFriendshipLinks(friendshipLinksId).then(res => {
            this.loading = true
            this.init()
          })
        }).catch(() => {
          // catch 为取消
          this.loading = false
        })
      },
      openFriendshipLinksDialog (id) {
        this.showEditFriendshipLinksDialog = true
        if (id !== null) {
          this.currentFriendshipLinksId = id
          this.friendshipLinksDialogTitle = 'Edit Carousel'
          this.friendshipLinksList.find(item => {
            if (item.id === this.currentFriendshipLinksId) {
              this.friendshipLinks.title = item.title
              this.friendshipLinks.visible = item.visible
              this.friendshipLinks.link = item.link
              this.friendshipLinks.order = item.order
              this.mode = 'edit'
            }
          })
        } else {
          this.friendshipLinksDialogTitle = 'Create Carousel'
          this.friendshipLinks.title = ''
          this.friendshipLinks.visible = true
          this.friendshipLinks.link = ''
          this.friendshipLinks.order = Math.max(...this.friendshipLinksList.map(o => o.order)) + 1
          this.mode = 'create'
        }
      },
      handleVisibleSwitch (row) {
        this.mode = 'edit'
        this.submitFriendshipLinks({
          id: row.id,
          title: row.title,
          link: row.link,
          order: row.order,
          visible: row.visible
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

  .visible-box {
    margin-top: 10px;
    width: 205px;
    float: left;
  }
</style>
