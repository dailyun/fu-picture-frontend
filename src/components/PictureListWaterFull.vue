<template>
  <Waterfall
    :list="dataList"
  >
  <template #default="{ item, url, index }">
    <div class="card">
      <!-- 图片展示 -->
      <LazyImg
        :url="item.thumbnailUrl?? item.url"
        :alt="item.name"
        :placeholder="null"
      />

      <!-- 文字内容 -->
      <div class="content">
        <h3 class="title">{{ item.name }}</h3>
        <p class="introduction">{{ item.introduction }}</p>

      </div>
    </div>
  </template>
  </Waterfall>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { LazyImg, Waterfall } from 'vue-waterfall-plugin-next'
import 'vue-waterfall-plugin-next/dist/style.css'

interface Props {
  dataList?: API.PictureVO[]
  loading?: boolean
  showOp?: boolean
  onReload?: () => void
}

const props = withDefaults(defineProps<Props>(), {
  dataList: () => [],
  loading: false,
  showOp: false,
})

console.log('完整数据项:', props.dataList)
console.log('第一个图片 URL:', props.dataList[0]?.url)
// 字段映射配置
const waterfallProp = computed(() => ({
  src: 'url',          // 将组件默认的 src 映射到你的 url 字段
  width: 'picWidth',   // 宽度字段映射
  height: 'picHeight'  // 高度字段映射
}))


</script>

<style scoped>
.card {
  border: 1px solid #eee;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0,0,0,0.05);
  transition: transform 0.2s;
}

.card:hover {
  transform: translateY(-2px);
}

.image {
  width: 100%;
  height: auto;
  aspect-ratio: 16/9; /* 根据实际需求调整 */
}

.content {
  padding: 1rem;
}

.title {
  font-size: 1.2rem;
  margin: 0 0 0.5rem;
  color: #333;
}

.introduction {
  font-size: 0.9rem;
  color: #666;
  margin: 0 0 0.5rem;
}

.meta {
  font-size: 0.8rem;
  color: #999;
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}
</style>
