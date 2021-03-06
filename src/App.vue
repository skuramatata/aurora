<template>
  <div id="app">
    <vue-progress-bar></vue-progress-bar>
    <Header />
    <div class="page">
      <Transition name="fade-transform" mode="out-in"> <RouterView /> </Transition>
    </div>
    <Footer @dropPanel="showPanel = true" />
    <Panel v-show="showPanel" @hidePanel="showPanel = false" />
    <div
      v-if="!$isMobile"
      @click="scrollTop"
      :style="backTopStyle"
      :class="['back-to-top', showBackTop && 'visible']"
    ></div>
  </div>
</template>

<script>
import Header from '@/components/Header'
import Footer from '@/components/Footer'
import Panel from '@/components/Panel'
import { on, getLocation } from '@/utils'

export default {
  name: 'App',
  components: {
    Header,
    Footer,
    Panel
  },
  data() {
    return {
      showBackTop: false,
      topDistance: -950,
      clientHeight: 0,
      lastScroll: new Date(),
      scrollTimer: '',
      showPanel: false
    }
  },
  computed: {
    backTopStyle() {
      return { top: `${this.topDistance}px` }
    }
  },
  created() {
    if (!this.$isMobile) {
      this.initProgress()
    }
    this.init()
  },
  mounted() {
    if (!this.$isMobile) {
      // 顶部进度条
      this.$Progress.finish()
      // 滚动页面
      on(window, 'scroll', () => this.handleScroll())
    }
  },
  methods: {
    // 注册顶部进度条
    initProgress() {
      this.$Progress.start()
      this.$router.beforeEach((to, from, next) => {
        this.$Progress.start()
        next()
      })
      this.$router.afterEach(() => {
        this.$Progress.finish()
      })
    },
    // 统计访客来源
    init() {
      const referrer = getLocation(document.referrer)
      const hostname = referrer.hostname || '直接访问'
      this.$store.dispatch('visitorStatistics', hostname)
    },
    // 滚动到顶部
    scrollTop() {
      this.$scroll(0)
    },
    // 监听页面滚动
    handleScroll(forced) {
      const now = new Date()
      if (now - this.lastScroll <= 100 && !forced) return
      if (!forced) {
        // 尾更新 task
        clearTimeout(this.scrollTimer)
        this.scrollTimer = setTimeout(() => {
          this.handleScroll(true)
        }, 300)
      }
      this.lastScroll = now

      // 更新顶部进度条
      const clientHeight = document.documentElement.clientHeight
      const scrollRange = document.body.clientHeight - clientHeight - 50
      const pageYOffset = window.pageYOffset
      this.$Progress.set((pageYOffset / scrollRange) * 100)

      // 判断位置，控制滚动到顶部
      const showBackTop = pageYOffset >= 200
      if (showBackTop !== this.showBackTop || this.clientHeight !== clientHeight) {
        this.showBackTop = showBackTop
        this.topDistance = -950 + (showBackTop ? clientHeight : 0)
        this.clientHeight = clientHeight
      }
    }
  }
}
</script>

<style lang="less" scope>
#app {
  position: relative;
  padding-bottom: 1rem;
  text-align: center;
  .back-to-top {
    position: fixed;
    top: 0;
    right: 0.25rem;
    width: 70px;
    height: 900px;
    z-index: 99;
    opacity: 1;
    transition: top 0.6s ease-out;
    background: url(https://i.loli.net/2019/01/24/5c496f5c617f6.png) no-repeat center;
    background-size: contain;
    will-change: left, top;
    &.visible {
      animation: float 3s linear infinite;
    }
  }
  .page {
    margin: 0 auto;
    padding: 0 0.12rem;
    max-width: 900px;
    user-select: text;
  }
}
</style>
