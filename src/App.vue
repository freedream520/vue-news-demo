<template>
  <div id="app">
    <view-box ref="ViewBox">
      <x-header slot="header" class="header">
        <span slot="overwrite-left"></span>
        <span>今日头条</span>
        <span slot="right"></span>
      </x-header>
      <sc class="scrolltab" :lock-y="true">
        <div class="tabwrap">
          <tab :line-width="1" active-color='#f85959'>
            <tab-item selected>推荐</tab-item>
            <tab-item>热点</tab-item>
            <tab-item>军事</tab-item>
            <tab-item>体育</tab-item>
            <tab-item>社会</tab-item>
            <tab-item>财经</tab-item>
            <tab-item>汽车</tab-item>
          </tab>
        </div>
      </sc>
      <scroller class="my-scroller" :on-refresh="refresh" :on-infinite="infinite" ref="myScroller">
        <swiper :list="bannerList" :loop="true" auto dots-class="bannerIndex"></swiper>
        <marquee class="my-marquee" direction="up">
          <marquee-item class="my-marquee-item" v-for="item in marqueeList"><a :href="item.url">{{item.title}}</a></marquee-item>
        </marquee>
        <panel :list="topList"></panel>
        <panel :list="moreData"></panel>
      </scroller>
    </view-box>
  </div>
</template>

<script>
import { ViewBox, XHeader, Scroller as Sc, Tab, TabItem, Swiper, Marquee, MarqueeItem, Panel } from 'vux'
export default {
  name: 'app',
  components: {
    ViewBox,
    XHeader,
    Sc,
    Tab,
    TabItem,
    Swiper,
    Marquee,
    MarqueeItem,
    Panel
  },
  data () {
    return {
      bannerList: [],
      marqueeList: [],
      topList: [],
      refreshKey: ['A', 'B01', 'B02', 'B03', 'B04', 'B05', 'B06', 'B07', 'B08', 'B09', 'B010'],
      moreData: [],
      KeyIndex: 0,
      initload: false,
      start: 0,
      end: 9
    }
  },
  created () {
    this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html').then(data => {
      this.bannerList = data.focus.filter(item => {  // banner
        return item.addata == null && item.picInfo.length
      }).map(item => {
        return {
          url: item.link,
          img: item.picInfo[0].url,
          title: item.title
        }
      })
      this.marqueeList = data.live.filter(item => {  // marqueeList
        return item.addata == null
      }).map(item => {
        return {
          title: item.title
        }
      })
      this.topList = data.list.filter(item => {  // topList 首屏新闻
        return item.addata == null && item.picInfo.length
      }).map(item => {
        return {
          url: item.link,
          src: item.picInfo[0].url,
          title: item.title,
          desc: item.source
        }
      })
      this.initload = true
    })
  },
  methods: {
    refresh () {
      this.KeyIndex ++
      if (this.KeyIndex === this.refreshKey.length - 1) {
        this.KeyIndex = 0
      }
      setTimeout(() => {
        this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html', {
          miss: '00',
          refresh: this.refreshKey[this.KeyIndex]
        }).then(data => {
          this.topList = data.list.filter(item => {
            return item.addata === null && item.picInfo.length
          }).map(item => {
            return {
              url: item.link,
              src: item.picInfo[0].url,
              title: item.title,
              desc: item.source
            }
          })
          this.$vux.toast.text(`已为您推荐${this.topList.length}条新闻`, 'top')
          this.$refs.myScroller.finishPullToRefresh()
        })
      }, 1000)
    },
    infinite () {
      if (!this.initload) {
        this.$refs.myScroller.finishInfinite()
        return
      }
      setTimeout(() => {
        this.$jsonp(`http://3g.163.com/touch/jsonp/sy/recommend/${this.start}-${this.end}.html`, {
          miss: '00',
          refresh: this.refreshKey[this.KeyIndex]
        }).then(data => {
          this.moreData = data.list.filter(item => {  // 更多新闻
            return item.addata === null && item.picInfo.length
          }).map(item => {
            return {
              url: item.link,
              src: item.picInfo[0].url,
              title: item.title,
              desc: item.source
            }
          })
          this.start = this.start + 10
          this.end = this.start + 10
          this.$refs.myScroller.finishInfinite()
        })
        this.$vux.toast.text('加载成功', 'bottom')
      }, 1000)
    }
  }
}
</script>
<style lang="less">
@import '~vux/src/styles/reset.less';
 html, body {  /*需要设置 html, body 高为100%:*/
  height: 100%;
  width: 100%;
  overflow: hidden;
}
#app {
  height: 100%;  /*view-box父div也需要为100%高度：*/
  .header{
    position: absolute;
    left: 0;
    top: 0;
    z-index: 99;
    width: 100%;
    background-color: #d43d3d;
    .vux-header-left{
      left: 14px;
      top: 10px;
      span{
        display: inline-block;
        width: 28px;
        height: 28px;
        background: url(assets/feed_ic_message_normal.png) no-repeat center 0;
        background-size: 28px auto;
        vertical-align: top;
      }
    }
    .vux-header-right{
      top: 10px;
      span{
        display: inline-block;
        width: 28px;
        height: 28px;
        background: url(assets/feed_ic_search_normal.png) no-repeat center 0;
        background-size: 28px auto;
        vertical-align: top;
      }
    }
  }
  .bannerIndex{
    i{
      &.active{
        background-color: #d43d3d;
      }
    }
  }
  .scrolltab{
    margin-top: 46px;
  }
  .tabwrap{
    width: 400px;
  }
  .my-marquee{
   margin: 10px 0;
  }
  .my-marquee-item{
    overflow: hidden;
    white-space: normal;
    text-overflow: ellipsis;
    padding-left: 10px;
    a{
      color: #999;
      display: inline-block;
      width: 100%;
    }
  }
  .weui-media-box__hd,  .weui-media-box__hd img {
        width: 102px;
        height: 78px;
    }

    .weui-media-box__bd {
       height: 78px;
    }

    .weui-media-box__title {
       white-space: normal;
       color: #404040;
       line-height: 24px;
    }
    .weui-media-box__desc{
      padding-top: 14px;
    }
    .loading-layer {
       padding-bottom: 40px;
    }
    .my-scroller {
        top: 90px;
     }
}
</style>
