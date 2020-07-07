<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <tab-control class="tab-control" :titles="['流行','新款','精选']"
                 @tabClick="tabClick" ref="tabControlFixed" v-show="isTabFixed"/>
    <scroll class="content" ref="scroll"
            :prob-type="3"
            @scroll="contentScroll"
            @pullingUp="loadMore"
            :pull-up-load="true"> <!--传入特定类型需要加冒号-->
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommends"></recommend-view>
      <feature-view/>
      <tab-control class="tab-control" :titles="['流行','新款','精选']"
                   @tabClick="tabClick" ref="tabControl"/>
      <goods-list :goods="showGoods"/>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
  import NavBar from 'components/common/navbar/NavBar'
  import TabControl from 'components/content/tabControl/TabControl'
  import GoodsList from 'components/content/goods/GoodsList'
  import Scroll from 'components/common/scroll/Scroll'
  import BackTop from 'components/content/backTop/BackTop'

  import HomeSwiper from './childComps/HomeSwiper'
  import RecommendView from './childComps/RecommendView'
  import FeatureView from './childComps/FeatureView'

  import { getHomeMultidata, getHomeGoods } from 'network/home'
  import { debounce } from 'common/utils'

  export default {
    name: 'Home',
    components: {
      NavBar,
      TabControl,
      GoodsList,
      Scroll,
      BackTop,

      HomeSwiper,
      RecommendView,
      FeatureView
    },
    data () {
      return {
        banners: [],
        recommends: [],
        goods: {
          'pop': { page: 0, list: [] },
          'new': { page: 0, list: [] },
          'sell': { page: 0, list: [] }
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed: false,
        saveY: 0
      }
    },
    created () {
      // 1.请求多个数据
      // 用this来调用当前模块methods内的方法，如果不加this，会调用import的同名方法
      this.getHomeMultidata()
      // 2. 请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
    },
    mounted () {
      // 监听  监听总线 里面的事件
      this.$bus.$on('itemImageLoad', () => {
        debounce(this.$refs.scroll.refresh, 500)
        // this.$refs.scroll.refresh() // scroll 插件在加载完数据后要刷新一下，让插件重新计算界面的高度，如果不刷新，可能会无法下拉
      })

    },
    computed: {
      showGoods () {
        return this.goods[this.currentType].list
      }
    },
    activated () {
      this.$refs.scroll.scrollTo(0, this.saveY)
      this.$refs.scroll.refresh()
    },
    deactivated () {
      this.saveY = this.$refs.scroll.scroll.y
    },
    methods: {

      /**
       * 事件监听
       */
      tabClick (index) {
        switch (index) {
          case 0:
            this.currentType = 'pop'
            break
          case 1:
            this.currentType = 'new'
            break
          case 2:
            this.currentType = 'sell'
            break
        }
        this.$refs.tabControl.currentIndex = index
        this.$refs.tabControlFixed.currentIndex = index
      },

      backClick () {
        this.$refs.scroll.scrollTo(0, 0, 1000)
      },
      contentScroll (pos) {
        // 判断显示back top
        this.isShowBackTop = pos.y < -1000

        // 判断tab control 吸顶
        this.isTabFixed = -pos.y > this.tabOffsetTop
      },
      /**
       * 网络请求相关方法
       */
      getHomeMultidata () {
        getHomeMultidata().then(res => {
          console.log(res)
          this.banners = res.data.banner.list
          this.recommends = res.data.recommend.list
        })
      },
      getHomeGoods (type) {
        // 每次取的值，在原来的页面上加1
        const page = this.goods[type].page + 1
        getHomeGoods(type, page).then(res => {
          console.log(res)
          const goodsList = res.data.list
          this.goods[type].list.push(...goodsList)
          this.goods[type].page += 1
        })
      },
      loadMore () {
        this.getHomeGoods(this.currentType)
        this.$refs.scroll.finishPullUp()
      },
      swiperImageLoad () {
        // 获取tabControl的offsetTop
        // tabControl赋值
        this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop
        console.log(this.tabOffsetTop)
      }
    }
  }
</script>

<style scoped>
  #home {
    padding-top: 44px;
    /*vh 视口高度*/
    height: 100vh;
    position: relative;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: white;

    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9;
  }

  /*.tab-control {*/
  /*  position: sticky;*/
  /*  top: 44px;*/
  /*  z-index: 10;*/
  /*}*/
  .tab-control {
    position: relative;
    z-index: 9;
  }

  .content {
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
  }
</style>
