<template>
  <Waterfall :list="list">
    <template #default="{ item, url, index,src }">
      <div class="card">
        <LazyImg :url=src />
        <p class="text">这是具体内容</p>
      </div>
    </template>
  </Waterfall>
</template>

<script setup lang="ts">
import { LazyImg, Waterfall } from 'vue-waterfall-plugin-next'
import 'vue-waterfall-plugin-next/dist/style.css'

import { ref } from 'vue'

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
// 转换为所需格式
let list = ref([]);
list = props.dataList.map((item) => ({
  src: item.url,
  title: item.name,
  category: item.category,
  tags: item.tags
}));
</script>

<style scoped></style>
