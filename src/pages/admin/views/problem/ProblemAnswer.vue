<template>
  <div class="Answer view">
    <Panel title="题解">
      <div class="list">
        <el-table
          v-loading="loading"
          element-loading-text="loading"
          ref="table"
          :data="answers"
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
            prop="last_update_time"
            label="LastUpdateTime">
            <template slot-scope="scope">
              {{scope.row.last_update_time | localtime }}
            </template>
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
              <icon-btn name="Edit" icon="edit" @click.native="openAnswerDialog(scope.row.id)"></icon-btn>
              <icon-btn name="Delete" icon="trash" @click.native="deleteAnswer(scope.row.id)"></icon-btn>
            </div>
          </el-table-column>
        </el-table>
        <div class="panel-options">
          <el-button type="primary" size="small" @click="openAnswerDialog(null)" icon="el-icon-plus">Create</el-button>
        </div>
      </div>
    </Panel>
    <!--对话框-->
    <el-dialog :title="answerDialogTitle" :visible.sync="showEditAnswerDialog"
               @open="onOpenEditDialog" :close-on-click-modal="false">
      <el-form label-position="top">
        <el-form-item :label="$t('m.Announcement_Title')" required>
          <el-input
            v-model="answer.title"
            :placeholder="$t('m.Announcement_Title')" class="title-input">
          </el-input>
        </el-form-item>
        <el-form-item :label="$t('m.Announcement_Content')" required>
          <Simditor v-model="answer.content"></Simditor>
        </el-form-item>
        <div class="visible-box">
          <span>{{$t('m.Announcement_visible')}}</span>
          <el-switch
            v-model="answer.visible"
            active-text=""
            inactive-text="">
          </el-switch>
        </div>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <cancel @click.native="showEditAnswerDialog = false"></cancel>
          <save type="primary" @click.native="submitAnswer"></save>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  import Simditor from '../../components/Simditor.vue'
  import api from '../../api.js'
  import {mapGetters} from 'vuex'

  export default {
    name: 'ProblemAnswer',
    components: {
      Simditor
    },
    computed: {
      ...mapGetters(['user', 'isSuperAdmin', 'isSecondary'])
    },
    data () {
      return {
        problemId: '',
        // 显示编辑题解对话框
        showEditAnswerDialog: false,
        // 题解列表
        answers: [],
        // 总题解数
        total: 0,
        // 当前题解id
        currentAnswerId: null,
        mode: 'create',
        // 题解 (new | edit) model
        answer: {
          title: '',
          visible: true,
          content: ''
        },
        // 对话框标题
        answerDialogTitle: 'Edit Answer',
        // 是否显示loading
        loading: true
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      init () {
        this.getAnswerList()
      },
      getAnswerList () {
        this.problemId = this.$route.params.problemId
        this.loading = true
        api.getProblemAnswerList(this.problemId).then(res => {
          this.loading = false
          this.answers = res.data.data
          this.total = res.data.data.total
        }, () => {
          this.loading = false
        })
      },
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getAnswerList(page)
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
      // 提交编辑
      // 默认传入MouseEvent
      submitAnswer (data = undefined) {
        let funcName = ''
        if (!data.title) {
          data = {
            id: this.currentAnswerId,
            title: this.answer.title,
            content: this.answer.content,
            visible: this.answer.visible,
            problem_id: this.problemId
          }
        }
        funcName = this.mode === 'edit' ? 'updateAnswer' : 'createAnswer'
        api[funcName](data).then(res => {
          this.showEditAnswerDialog = false
          this.init()
        }).catch()
      },
      // 删除题解
      deleteAnswer (AnswerId) {
        this.$confirm('Are you sure you want to delete this Answer?', 'Warning', {
          confirmButtonText: 'Delete',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          // then 为确定
          this.loading = true
          api.deleteAnswer(AnswerId).then(res => {
            this.loading = true
            this.init()
          })
        }).catch(() => {
          // catch 为取消
          this.loading = false
        })
      },
      openAnswerDialog (id) {
        this.showEditAnswerDialog = true
        if (id !== null) {
          this.currentAnswerId = id
          this.answerDialogTitle = 'Edit Answer'
          this.answers.find(item => {
            if (item.id === this.currentAnswerId) {
              this.answer.title = item.title
              this.answer.visible = item.visible
              this.answer.content = item.content
              this.mode = 'edit'
            }
          })
        } else {
          this.answerDialogTitle = 'Create Answer'
          this.answer.title = ''
          this.answer.visible = true
          this.answer.content = ''
          this.mode = 'create'
        }
      },
      handleVisibleSwitch (row) {
        this.mode = 'edit'
        this.submitAnswer({
          id: row.id,
          title: row.title,
          content: row.content,
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
