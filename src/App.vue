<template>
  <div id="app">
    <view-box>
      <x-header slot="header" :left-options="{showBack: false, backText: 'Back'}">
        <div slot="left">返回</div>
        <div>X-news</div>
        <div slot="right">搜索</div>
      </x-header>

      <scroller :lock-y="true">
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
      </scroller>

      <swiper :list="swiperList" v-model="swiperIndex" :loop="true"></swiper>

      <marquee class="marquee">
        <marquee-item v-for="item in marqueeList">{{item.title}}</marquee-item>
      </marquee>

      <panel :list="dataList">

      </panel>

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
import { ViewBox, XHeader, Tabbar, TabbarItem, Scroller, Tab, TabItem, Swiper, Marquee, MarqueeItem, Panel } from "vux";

export default {
  name: 'app',
  data: function() {
    return {
      swiperList: [],
      swiperIndex: 0,
      marqueeList: [],
      dataList: [],
    };
  },
  components: {
    ViewBox,
    XHeader,
    Tabbar,
    TabbarItem,
    Scroller,
    Tab,
    TabItem,
    Swiper,
    Marquee,
    MarqueeItem,
    Panel
  },
  created() {
    this.$jsonp("http://3g.163.com/touch/jsonp/sy/recommend/0-9.html")
      .then( data => {
        console.log(data);
        this.swiperList = data.focus.filter(item => {
          return item.addata === null;
        }).map(item => {
          return {
            url: item.link,
            img: item.picInfo[0].url,
            title: item.title
          }
        });

        this.dataList = data.list.filter(item => {
          return item.addata === null;
        }).map(item => {
          return {
            src: item.picInfo[0].url,
            title: item.title,
            desc: item.digest,
            url: item.link,
          }
        });

        this.marqueeList = data.live.filter(item => {
          return item.addata === null;
        }).map(item => {
          return {
            title: item.title
          }
        })
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
  }
</style>
