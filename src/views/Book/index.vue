<template>
  <div id="book">
    <Transition name="fade-transform" mode="out-in">
      <div class="card" v-if="book.length">
        <Quote :quote="$config.bookOpts.qoute" />
        <ul class="content">
          <li v-for="item in book" :key="item.name">
            <div class="info">
              <img :src="item.cover" alt />
              <div>
                <h3>{{ item.name }}</h3>
                <p>作者：{{ item.author }}</p>
                <p>出版时间：{{ item.published }}</p>
                <p>阅读进度：{{ item.progress }}</p>
                <p>
                  读书笔记：
                  <a v-if="item.postLink" :href="item.postLink" rel="noopener noreferrer" target="_blank">
                    {{ item.postTitle }}
                  </a>
                  <span v-else>暂无</span>
                </p>
                <p>
                  推荐指数：
                  <i class="icon icon-star" v-for="i in parseInt(item.rating)" :key="`star-${i}`"></i>
                  <i
                    class="icon icon-star unstar"
                    v-for="i in 5 - parseInt(item.rating)"
                    :key="`unstar-${i}`"
                  ></i>
                </p>
              </div>
            </div>
            <div class="description">{{ item.description }}</div>
          </li>
        </ul>
      </div>
      <Loading v-else />
    </Transition>

    <Comment v-if="$config.bookOpts.enableComment && initComment" title="书单" />
  </div>
</template>

<script>
import Loading from '@/components/Loading'
import Quote from '@/components/Quote'
import Comment from '@/components/Comment'

export default {
  name: 'Book',
  components: {
    Loading,
    Quote,
    Comment
  },
  data() {
    return {
      book: [],
      initComment: false
    }
  },
  async created() {
    await this.queryBooks()
    this.initComment = true
  },
  methods: {
    // 获取书单
    async queryBooks() {
      this.book = await this.$store.dispatch('queryPage', { type: 'book' })
    }
  }
}
</script>

<style lang="less" scoped>
@import './index.less';
</style>
