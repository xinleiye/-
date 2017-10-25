<template>
  <div id="app">
    <view-box>
      <x-header slot="header" :left-options="{showBack: false, backText: 'Back'}">
        <div slot="left">返回</div>
        <div>X-news</div>
        <div slot="right">搜索</div>
      </x-header>

      <sc :lock-y="true">
        <div class="tab-width">
          <tab>
            <tab-item selected>推荐</tab-item>
            <tab-item>时事</tab-item>
            <tab-item>科技</tab-item>
            <tab-item>体育</tab-item>
            <tab-item>汽车</tab-item>
            <tab-item>娱乐</tab-item>
          </tab>
        </div>
      </sc>

      <scroller class="sc-content" :on-refresh="refresh" :on-infinite="infinite" ref="newref">
        <swiper :list="swiperList" v-model="swiperIndex" :loop="true"></swiper>

        <marquee class="marquee">
          <marquee-item v-for="item in marqueeList">{{item.title}}</marquee-item>
        </marquee>

        <panel :list="dataList"></panel>
        <panel :list="moreDataList"></panel>

      </scroller>

      <tabbar slot="bottom">
        <tabbar-item>
          <img slot="icon" src="./assets/icon_nav_cell.png" />
          <span slot="label">首页</span>
        </tabbar-item>
        <tabbar-item>
          <img slot="icon" src="./assets/icon_nav_article.png" />
          <span slot="label">推荐</span>
        </tabbar-item>
        <tabbar-item>
          <img slot="icon" src="./assets/icon_nav_msg.png" />
          <span slot="label">热门</span>
        </tabbar-item>
      </tabbar>
    </view-box>
    <router-view></router-view>
  </div>
</template>

<script>
import { ViewBox, XHeader, Tabbar, TabbarItem, Scroller as Sc, Tab, TabItem, Swiper, Marquee, MarqueeItem, Panel } from "vux";

// 上拉、下拉时，获取不同新闻的标记
let newsKey = "A";
const refreshKey = ['A', 'B01', 'B02', 'B03', 'B04', 'B05', 'B06', 'B07', 'B08', 'B09', 'B010'];
let refreshKeyIndex = -1;

// 新获取新闻的数量
let start = 0;
let end = 9;

// 防止页面初始化时，触发获取新闻的动作
let firstLoaded = false;

function getNewsKey() {
  refreshKeyIndex++;
  if (refreshKeyIndex >= refreshKey.length) {
    refreshKeyIndex = -1;
  }
  newsKey = refreshKey[refreshKeyIndex];
}

export default {
  name: 'app',
  data: function() {
    return {
      swiperList: [],
      swiperIndex: 0,
      marqueeList: [],
      dataList: [],
      moreDataList: []
    };
  },
  components: {
    ViewBox,
    XHeader,
    Tabbar,
    TabbarItem,
    Sc,
    Tab,
    TabItem,
    Swiper,
    Marquee,
    MarqueeItem,
    Panel
  },
  methods: {
    refresh() {
      getNewsKey();
      //下拉时获取新闻数据
      this.$jsonp("http://3g.163.com/touch/jsonp/sy/recommend/0-9.html", {
        miss: "00",
        refresh: newsKey
      }).then( data => {
        var tmpDataList = [];
        tmpDataList = data.list.filter(item => {
          return item.addata === null && item.picInfo.length;
        }).map(item => {
          return {
            src: item.picInfo[0].url,
            title: item.title,
            desc: item.digest,
            url: item.link,
          }
        });
        this.dataList = tmpDataList.concat(this.dataList);
        this.$refs.newref.finishPullToRefresh();
        this.$vux.toast.text(`成功为您推荐${this.dataList.length}条新闻`, "top");
      });
    },
    infinite() {

      if (!firstLoaded) {
        this.$refs.newref.finishInfinite();
        return;
      }
      // 上拉时，获取新闻数据
      this.$jsonp(`http://3g.163.com/touch/jsonp/sy/recommend/${start}-${end}.html`, {
        miss: "00",
        refresh: newsKey
      }).then( data => {
        setTimeout(() => {
          var tmpDataList;
          tmpDataList = data.list.filter(item => {
            return item.addata === null && item.picInfo.length;
          }).map(item => {
            return {
              src: item.picInfo[0].url,
              title: item.title,
              desc: item.digest,
              url: item.link,
            }
          });
          this.moreDataList = this.moreDataList.concat(tmpDataList);
          start += 10;
          end = start + 9;
          this.$refs.newref.finishInfinite();
          this.$vux.toast.text(`成功为您推荐${this.dataList.length}条新闻`, "top");
        }, 1000);
      });
    }
  },
  created() {
    this.$jsonp("http://3g.163.com/touch/jsonp/sy/recommend/0-9.html")
      .then( data => {
        //提取幻灯片新闻数据
        this.swiperList = data.focus.filter(item => {
          return item.addata === null && item.picInfo.length;
        }).map(item => {
          return {
            url: item.link,
            img: item.picInfo[0].url,
            title: item.title
          }
        });
        //提取首页新闻数据
        this.dataList = data.list.filter(item => {
          return item.addata === null && item.picInfo.length;
        }).map(item => {
          return {
            src: item.picInfo[0].url,
            title: item.title,
            desc: item.digest,
            url: item.link,
          }
        });
        //提取跑马灯新闻数据
        this.marqueeList = data.live.filter(item => {
          return item.addata === null && item.picInfo.length;
        }).map(item => {
          return {
            title: item.title
          }
        });
        firstLoaded = true;
    });
  }
}
</script>

<style lang="less">
  @import "~vux/src/styles/reset.less";

  html,body {
    margin: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  #app {
    height: 100%;
    .tab-width {
      width: 200%;
    }
    .marquee {
      margin: 10px;
    }
    .sc-content {
      margin-top: 90px;
    }
    .loading-layer {
      padding-bottom: 80px;
    }
    .weui-media-box__hd, .weui-media-box__hd img {
      width: 102px;
      height: 78px;
    }
    .weui-media-box__bd {
      height: 78px;
    }
  }
</style>
