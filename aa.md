我有一个看小说的网页地址 我需要手动点击下一页才能翻页  有什么办法 可以监听我的滚动条，滚到最地下，自动加载下一页的技术吗



用vue帮我实现这个功能，

 




功能包括：

1.无限滚动加载：当用户滚动接近底部时自动加载下一章或下一页，通过监听scroll事件，使用节流函数优化性能。
2.内容清理：从获取的HTML中提取主要内容，移除不必要的标签和属性，保证内容干净。
3.加载状态提示：显示加载中的旋转图标和文字，使用过渡动画效果。
4.章节分页管理：处理章节和页数的递增，超过3页则切换章节。
5.错误处理和重试：失败时重试，超过次数标记结束，并回滚页码。
6.结束标记：全部加载完成后显示提示。
7.滚动位置判断：根据滚动阈值判断是否需要加载，避免重复加载。
step0:  npm install lodash

step1：C:\Users\wangrusheng\PycharmProjects\untitled18\src\views\Home.vue

```typescript
<template>
  <div
    class="container"
    @scroll="handleScroll"
    ref="container"
  >
    <div class="content" v-html="processedContent"></div>
    <transition name="fade">
      <div v-if="loading" class="loading-indicator">
        <div class="spinner"></div>
        正在加载下一章...
      </div>
    </transition>
    <div v-if="isEnd" class="end-marker">已阅读全部内容</div>
  </div>
</template>

<script>
import { throttle } from 'lodash'

export default {
  data() {
    return {
      processedContent: '',
      currentChapter: 80,  // 初始章节
      currentPage: 1,
      loading: false,
      isEnd: false,
      lastScrollPos: 0,
      errorCount: 0,
      maxErrorRetry: 3,
      throttleDelay: 300,
      scrollThreshold: 150
    }
  },

  mounted() {
    this.initScroll()
    this.loadContent()
  },

  beforeDestroy() {
    window.removeEventListener('scroll', this.throttledHandler)
  },

  methods: {
    initScroll() {
      this.throttledHandler = throttle(this.checkScroll, this.throttleDelay)
      window.addEventListener('scroll', this.throttledHandler, { passive: true })
    },

    async loadContent() {
      if (this.isEnd || this.loading) return

      this.loading = true
      try {
        const url = this.buildChapterUrl()
        const response = await fetch(url)

        if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`)

        const html = await response.text()
        if (this.isLastPage(html)) {
          this.isEnd = true
          return
        }

        this.processedContent += this.cleanContent(html)
        this.currentPage++
        this.errorCount = 0

      } catch (error) {
        console.error('加载失败:', error)
        if (++this.errorCount >= this.maxErrorRetry) {
          this.isEnd = true
        }
        this.rollbackPage()
      } finally {
        this.loading = false
      }
    },

    buildChapterUrl() {
      const base = `https://m.1qxs.com/xs/64584/${this.currentChapter}`
      return this.currentPage > 1 ? `${base}/${this.currentPage}` : base
    },

    cleanContent(html) {
      const parser = new DOMParser()
      const doc = parser.parseFromString(html, 'text/html')
      const content = doc.querySelector('.content') || doc.body

      // 清理不需要的元素
      const removables = [
        'script', 'style', 'img', 'iframe',
        '.ad', '.advertisement', '.comments'
      ]
      removables.forEach(selector => {
        content.querySelectorAll(selector).forEach(el => el.remove())
      })

      // 简化内容样式
      content.querySelectorAll('*').forEach(el => {
        el.removeAttribute('style')
        el.removeAttribute('class')
      })

      return content.innerHTML
    },

    checkScroll() {
      const currentScroll = window.scrollY + window.innerHeight
      const pageHeight = document.documentElement.scrollHeight

      if (pageHeight - currentScroll < this.scrollThreshold &&
          this.lastScrollPos < window.scrollY) {
        this.handleNextPage()
      }
      this.lastScrollPos = window.scrollY
    },

    handleNextPage() {
      if (this.currentPage > 3) {
        this.currentChapter++
        this.currentPage = 1
      }
      this.loadContent()
    },

    isLastPage(html) {
      // 根据实际页面内容判断章节结束
      return html.includes('本章已完') ||
             html.includes('未完待续') ||
             html.includes('下一章')
    },

    rollbackPage() {
      if (this.currentPage > 1) {
        this.currentPage--
      } else if (this.currentChapter > 80) {
        this.currentChapter--
        this.currentPage = 3
      }
    },

    handleScroll() {
      // 保留原生滚动处理
      this.throttledHandler()
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
}

.content {
  line-height: 1.8;
  font-size: 18px;
  color: #333;
}

.content >>> p {
  margin: 1em 0;
  text-indent: 2em;
}

.loading-indicator {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  border-radius: 25px;
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 14px;
}

.spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255,255,255,0.3);
  border-radius: 50%;
  border-top-color: #fff;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

.end-marker {
  text-align: center;
  padding: 30px;
  color: #666;
  font-size: 16px;
}
</style>
```

end