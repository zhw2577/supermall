<template>
  <div class="wrapper" ref="wrapper">
    <div class="content">
      <slot></slot>
    </div>
  </div>
</template>

<script>
  import BScroll from 'better-scroll'

  export default {
    name: 'Scroll',
    data () {
      return {
        scroll: null
      }
    },
    props: {
      probType: {
        type: Number,
        default: 0,
      },
      pullUpLoad: {
        type: Boolean,
        default: false
      }

    },
    mounted () {
      this.scroll = new BScroll(this.$refs.wrapper, {
        probType: this.probType,
        click: true,
        pullUpLoad: this.pullUpLoad,
        mouseWheel: true,//开启鼠标滚轮
        disableMouse: false,//启用鼠标拖动
        disableTouch: false//启用手指触摸

      })

      if (this.probType >= 2) {
        // 监听滚动位置
        this.scroll.on('scroll', (pos) => {
          this.$emit('scroll', pos)
        })
      }

      if (this.pullUpLoad) {
        // 监听上拉事件
        this.scroll.on('pullingUp', () => {
          this.$emit('pullingUp')
        })
      }
    },
    methods: {
      scrollTo (x, y, time = 500) {
        this.scroll.scrollTo(x, y, time)
      },
      finishPullUp () {
        this.scroll.finishPullUp()
      },
      refresh () {
        this.scroll.refresh()
      },
    }
  }
</script>

<style scoped>

</style>
