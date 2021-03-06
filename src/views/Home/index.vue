<template>
  <div id="home">
    <Transition name="fade-transform" mode="out-in">
      <div class="main" v-if="posts.length">
        <article
          class="card"
          data-aos="fade-up"
          v-for="post in posts"
          :key="post.id"
          @click="gotoPost(post.number)"
          @mouseenter="showTips(post)"
        >
          <div class="post-header">
            <Lazyload :src="post.cover.src" :alt="post.cover.title" />
            <div>
              <h3>{{ post.title }}</h3>
              <span>{{ post.cover.title }}</span>
            </div>
          </div>
          <div class="post-body"><MarkDown :content="post.description" :onlyRender="true" /></div>
          <div class="post-meta">
            <span> <i class="icon icon-calendar"></i> {{ post.created_at }} </span>
            <span> <i class="icon icon-fire"></i> 热度{{ post.times || 1 }}℃ </span>
            <span>
              <i class="icon icon-bookmark-empty"></i> {{ post.milestone ? post.milestone.title : '未分类' }}
            </span>
            <span>
              <i class="icon icon-tag"></i>
              <span v-for="label in post.labels.slice(0, 2)" :key="label.id">{{ label.name }}</span>
            </span>
          </div>
        </article>
      </div>
    </Transition>

    <Transition name="fade-transform" mode="out-in">
      <div v-if="!list.length"><Loading /></div>
      <div class="btn-group" v-if="list.length && (!isDisabledPrev || !isDisabledNext)">
        <Pagination
          :loading="loading"
          :isDisabledPrev="isDisabledPrev"
          :isDisabledNext="isDisabledNext"
          @handleClick="queryPosts"
        />
      </div>
    </Transition>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import AOS from 'aos'
import SmoothScroll from 'smooth-scroll'
import MarkDown from '@/components/MarkDown'
import Loading from '@/components/Loading'
import Lazyload from '@/components/Lazyload'
import Pagination from '@/components/Pagination'

export default {
  name: 'Home',
  components: {
    MarkDown,
    Loading,
    Lazyload,
    Pagination
  },
  data() {
    return {
      loading: false,
      page: 0,
      pageSize: 12,
      maxPage: 0,
      posts: [],
      list: [],
      scroll: new SmoothScroll()
    }
  },
  computed: mapState({
    isDisabledPrev() {
      return this.page <= 1
    },
    isDisabledNext() {
      if (!this.maxPage) return false
      return this.page >= this.maxPage
    }
  }),
  async created() {
    await this.queryPosts()

    AOS.init({
      duration: 2000,
      easing: 'ease-out',
      debounceDelay: 200,
      offset: 50
    })
  },
  methods: {
    // 获取文章列表
    async queryPosts(type = 'next') {
      if (this.loading) return
      const queryPage = type === 'prev' ? this.page - 1 : this.page + 1

      if (this.list[queryPage]) {
        this.scrollTop(() => {
          this.page = queryPage
          this.posts = this.list[queryPage]
        })
        return
      }

      this.loading = true
      const posts = await this.$store.dispatch('queryPosts', {
        page: queryPage,
        pageSize: this.pageSize
      })
      this.loading = false
      if (posts.length === 0) {
        this.maxPage = queryPage - 1
        return
      }
      this.scrollTop(() => {
        this.page = queryPage
        this.posts = posts
        this.$set(this.list, queryPage, posts)
        if (posts.length < this.pageSize) {
          this.maxPage = queryPage
        }
        // 获取文章热度
        this.$nextTick(async () => {
          const ids = this.posts.map(o => o.id)
          const hot = await this.$store.dispatch('queryHot', { ids })
          this.posts = this.posts.map((o, i) => {
            o.times = hot[i]
            return o
          })
        })
      })
    },
    // 滚动到顶部
    scrollTop(callback) {
      this.$nextTick(() => {
        this.$scroll(0)
        // 对于移动端延长滚动时间
        const delayTime = this.$isMobile ? 500 : 0
        setTimeout(callback, 1000 + delayTime)
        setTimeout(AOS.refresh, 1500 + delayTime)
      })
    },
    // 跳转文章页
    gotoPost(number) {
      this.$router.push({ name: 'post', params: { number } })
    },
    // 看板娘
    showTips(post) {
      const tips = `要去看看<span style="color: #b854d4"> ${post.title} </span>吗？`
      this.$store.dispatch('showTips', { tips })
    }
  }
}
</script>

<style lang="less" scope>
@import './index.less';
</style>
