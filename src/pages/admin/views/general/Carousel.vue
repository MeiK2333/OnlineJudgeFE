<template>
  <div class="announcement view">
    <Panel title="轮播图管理">
      <div class="list">
        <el-table
          v-loading="loading"
          element-loading-text="loading"
          ref="table"
          :data="carouselList"
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
              <icon-btn name="Edit" icon="edit" @click.native="openCarouselDialog(scope.row.id)"></icon-btn>
              <icon-btn name="Delete" icon="trash" @click.native="deleteCarousel(scope.row.id)"></icon-btn>
            </div>
          </el-table-column>
        </el-table>
        <div class="panel-options">
          <el-button type="primary" size="small" @click="openCarouselDialog(null)" icon="el-icon-plus">Create</el-button>
        </div>
      </div>
    </Panel>
    <!--对话框-->
    <el-dialog :title="carouselDialogTitle" :visible.sync="showEditCarouselDialog"
               @open="onOpenEditDialog" :close-on-click-modal="false">
      <el-form label-position="top">
        <el-form-item label="图片" required>
          <el-upload
            class="upload-demo"
            action="/api/admin/upload_image/"
            name="image"
            :data="{original_filename: carousel.title}"
            :on-success="uploadImageSuccess"
            :show-file-list="false"
          >
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件</div>
          </el-upload>
          <img v-if="carousel.file_path" :src="carousel.file_path" style="max-width: 100%" />
        </el-form-item>
        <el-form-item label="标题" required>
          <el-input
            v-model="carousel.title"
            placeholder="标题" class="title-input">
          </el-input>
        </el-form-item>
        <el-form-item label="排序优先级（越高越靠前）" required>
          <el-input-number
            v-model="carousel.order"
            placeholder="排序优先级（越高越靠前）" class="title-input">
          </el-input-number>
        </el-form-item>
        <div class="visible-box">
          <span>是否可见</span>
          <el-switch
            v-model="carousel.visible"
            active-text=""
            inactive-text="">
          </el-switch>
        </div>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <cancel @click.native="showEditCarouselDialog = false"></cancel>
          <save type="primary" @click.native="submitCarousel"></save>
        </span>
    </el-dialog>
  </div>
</template>

<script>
  import Simditor from '../../components/Simditor.vue'
  import api from '../../api.js'
  import {mapGetters} from 'vuex'

  export default {
    name: 'Carousel',
    components: {
      Simditor
    },
    computed: {
      ...mapGetters(['user', 'isSuperAdmin', 'isSecondary'])
    },
    data () {
      return {
        showEditCarouselDialog: false,
        carouselList: [],
        currentCarouselId: null,
        mode: 'create',
        carousel: {
          title: '',
          visible: true,
          file_path: '',
          order: 0
        },
        // 对话框标题
        carouselDialogTitle: 'Edit Carousel',
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
        this.getCarouselList()
      },
      uploadImageSuccess (response, file, fileList) {
        this.carousel.file_path = response.file_path
      },
      getCarouselList () {
        this.loading = true
        api.getCarouselList().then(res => {
          this.loading = false
          this.carouselList = res.data.data
        }, res => {
          this.loading = false
        })
      },
      // 切换页码回调
      currentChange (page) {
        this.currentPage = page
        this.getAnnouncementList(page)
      },
      getAnnouncementList (page) {
        this.loading = true
        api.getAnnouncementList((page - 1) * this.pageSize, this.pageSize).then(res => {
          this.loading = false
          this.total = res.data.data.total
          this.announcementList = res.data.data.results
        }, res => {
          this.loading = false
        })
      },
      getContestAnnouncementList () {
        this.loading = true
        api.getContestAnnouncementList(this.contestID).then(res => {
          this.loading = false
          this.announcementList = res.data.data
        }).catch(() => {
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
      submitCarousel (data = undefined) {
        if (!data.title) {
          data = {
            id: this.currentCarouselId,
            title: this.carousel.title,
            file_path: this.carousel.file_path,
            order: this.carousel.order,
            visible: this.carousel.visible
          }
        }
        let func = this.mode === 'edit' ? api.updateCarousel : api.createCarousel
        func(data).then(res => {
          this.showEditCarouselDialog = false
          this.init()
        })
      },
      deleteCarousel (carouselId) {
        this.$confirm('Are you sure you want to delete this carousel?', 'Warning', {
          confirmButtonText: 'Delete',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          // then 为确定
          this.loading = true
          api.deleteCarousel(carouselId).then(res => {
            this.loading = true
            this.init()
          })
        }).catch(() => {
          // catch 为取消
          this.loading = false
        })
      },
      openCarouselDialog (id) {
        this.showEditCarouselDialog = true
        if (id !== null) {
          this.currentCarouselId = id
          this.carouselDialogTitle = 'Edit Carousel'
          this.carouselList.find(item => {
            if (item.id === this.currentCarouselId) {
              this.carousel.title = item.title
              this.carousel.visible = item.visible
              this.carousel.file_path = item.file_path
              this.carousel.order = item.order
              this.mode = 'edit'
            }
          })
        } else {
          this.carouselDialogTitle = 'Create Carousel'
          this.carousel.title = ''
          this.carousel.visible = true
          this.carousel.file_path = ''
          this.carousel.order = 0
          this.mode = 'create'
        }
      },
      handleVisibleSwitch (row) {
        this.mode = 'edit'
        this.submitCarousel({
          id: row.id,
          title: row.title,
          file_path: row.file_path,
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
