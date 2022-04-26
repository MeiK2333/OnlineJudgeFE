<template>
  <Row type="flex" justify="space-around">
    <Col :span="22">
    <div>
      <Panel shadow :padding="10">
        <div slot="title">
          {{$t('m.ProblemAnswer')}}
        </div>
        <div slot="extra">
          <ul class="filter">
            <template  v-if="listVisible">
            </template>
            <template v-else>
              <li>
                <Button type="ghost" icon="ios-undo" @click="goBack">{{$t('m.Back')}}</Button>
              </li>
            </template>
          </ul>
        </div>

        <transition-group name="announcement-animate" mode="in-out">
          <div class="no-announcement" v-if="!answers.length" key="no-announcement">
            <p>{{$t('m.No_Answers')}}</p>
          </div>
          <template v-if="listVisible">
            <ul class="announcements-container" key="list">
              <li v-for="answer in answers" :key="answer.title">
                <div class="flex-container">
                  <div class="title"><a class="entry" @click="goAnswer(answer)">
                    {{answer.title}}</a></div>
                  <div class="date">{{answer.create_time | localtime }}</div>
                  <div class="creator"> {{$t('m.By')}} {{answer.created_by.username}}</div>
                </div>
              </li>
            </ul>
          </template>

          <template v-else>
            <div v-katex v-html="answer.content" key="content" class="content-container markdown-body"></div>
          </template>
        </transition-group>
      </Panel>
    </div>
    </Col>
  </Row>
</template>

<script>
  import api from '@oj/api'
  import time from '@/utils/time'

  export default {
    name: 'problem-answer',
    data () {
      return {
        listVisible: true,
        index: 0,
        answer: '',
        answers: [],
        problemID: ''
      }
    },
    mounted () {
      this.getAnswerList()
    },
    methods: {
      getAnswerList () {
        this.problemID = this.$route.params.problemID
        this.btnLoading = true
        api.getProblemAnswerList(this.problemID).then(res => {
          this.btnLoading = false
          this.answers = res.data.data
          this.total = res.data.data.total
        }, () => {
          this.btnLoading = false
        })
      },
      getDuration (startTime, endTime) {
        return time.duration(startTime, endTime)
      },
      goBack () {
        this.answer = ''
        this.listVisible = true
      },
      goAnswer (answer) {
        this.answer = answer
        this.listVisible = false
      }
    }
  }
</script>

<style lang="less" scoped>
  .contest {
    &-title {
      font-style: italic;
      font-size: 21px;
    }
    &-content {
      padding: 0 70px 40px 70px;
      &-description {
        margin-top: 25px;
      }
    }
  }

  .announcement {
    margin-top: 20px;
  }
  .announcements-container {
    margin-top: -10px;
    margin-bottom: 10px;
    li {
      padding-top: 15px;
      list-style: none;
      padding-bottom: 15px;
      margin-left: 20px;
      font-size: 16px;
      border-bottom: 1px solid rgba(187, 187, 187, 0.5);
      &:last-child {
        border-bottom: none;
      }
      .flex-container {
        .title {
          flex: 1 1;
          text-align: left;
          padding-left: 10px;
          a.entry {
            color: #495060;
            &:hover {
              color: #2d8cf0;
              border-bottom: 1px solid #2d8cf0;
            }
          }
        }
        .creator {
          flex: none;
          width: 200px;
          text-align: center;
        }
        .date {
          flex: none;
          width: 200px;
          text-align: center;
        }
      }
    }
  }

  .content-container {
    padding: 0 20px 20px 20px;
  }

  .no-announcement {
    text-align: center;
    font-size: 16px;
  }changeLocale

   .announcement-animate-enter-active {
     animation: fadeIn 1s;
   }
</style>
