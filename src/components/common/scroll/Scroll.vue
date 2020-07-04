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
    mounted () {
      console.log(this.$refs.wrapper)
      // let wrapper = document.querySelector('.wrapper')
      // this.scroll = new BScroll(wrapper)
      this.scroll = new BScroll(this.$refs.wrapper, {
        click: true,
        mouseWheel: true,//开启鼠标滚轮
        disableMouse: false,//启用鼠标拖动
        disableTouch: false//启用手指触摸
      })
    }
  }
</script>

<style scoped>

</style>
