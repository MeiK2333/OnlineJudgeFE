<template>
<div>
  <Row type="flex" justify="space-around">
    <Col :span="22">
    <Carousel v-if="carousels.length" v-model="idx" trigger="hover" autoplay :autoplay-speed="6000" class="contest" style="text-align: center; margin-bottom: 20px">
      <CarouselItem v-for="(carousel, index) of carousels" :key="index">
        <img :src="carousel.file_path"  :alt="carousel.title" style="max-width: 100%"/>
      </CarouselItem>
    </Carousel>
    </Col>
  </Row>
  <Row type="flex" justify="space-around">
    <Col :span="19" style="padding-left: 9px; padding-right: 9px;">
      <panel shadow v-if="contests.length">
      <div slot="title">
        <Button type="text"  class="contest-title" @click="goContest">{{contests[index].title}}</Button>
      </div>
      <Carousel v-model="index" trigger="hover" autoplay :autoplay-speed="6000" class="contest">
        <CarouselItem v-for="(contest, index) of contests" :key="index">
          <div class="contest-content">
            <div class="contest-content-tags">
              <Button type="info" shape="circle" size="small" icon="calendar">
                {{contest.start_time | localtime('YYYY-M-D HH:mm') }}
              </Button>
              <Button type="success" shape="circle" size="small" icon="android-time">
                {{getDuration(contest.start_time, contest.end_time)}}
              </Button>
              <Button type="warning" shape="circle" size="small" icon="trophy">
                {{contest.rule_type}}
              </Button>
            </div>
            <div class="contest-content-description">
              <blockquote v-html="contest.description"></blockquote>
            </div>
          </div>
        </CarouselItem>
      </Carousel>
    </panel>
    <Announcements class="announcement"></Announcements>
    </Col>

    <Col v-if="links.length" :span="5" style="padding-left: 9px; padding-right: 9px;">
      <Panel :padding="10">
        <div slot="title" class="taglist-title">{{$t('m.Links')}}</div>
        <a class="friendship-link" v-for="(link, index) of links" :key="index" :href="link.link" target="_blank">{{link.title}}</a>
      </Panel>
    </Col>
  </Row>
</div>
</template>

<script>
  import Announcements from './Announcements.vue'
  import api from '@oj/api'
  import time from '@/utils/time'
  import { CONTEST_STATUS } from '@/utils/constants'

  export default {
    name: 'home',
    components: {
      Announcements
    },
    data () {
      return {
        contests: [],
        index: 0,
        idx: 0,
        carousels: [],
        links: []
      }
    },
    mounted () {
      let params = {status: CONTEST_STATUS.NOT_START}
      api.getContestList(0, 5, params).then(res => {
        this.contests = res.data.data.results
      })
      this.getCarousel()
      this.getLinks()
    },
    methods: {
      getDuration (startTime, endTime) {
        return time.duration(startTime, endTime)
      },
      getCarousel () {
        api.getCarousel().then(res => {
          this.carousels = res.data.data
        })
      },
      getLinks () {
        api.getLinks().then(res => {
          this.links = res.data.data
        })
      },
      goContest () {
        this.$router.push({
          name: 'contest-details',
          params: {contestID: this.contests[this.index].id}
        })
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
    //margin-top: 20px;
  }

  .friendship-link {
    padding-left: 3px;
    padding-right: 3px;
  }

  .taglist-title {
    margin-left: -10px;
    margin-bottom: -10px;
  }
</style>
