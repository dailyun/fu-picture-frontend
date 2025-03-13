<template>
  <div id="homePage">
    <!-- 搜索框 -->
    <div class="search-bar">
      <a-input-search
        placeholder="搜搜看"
        v-model:value="searchParams.searchText"
        enter-button="搜索"
        size="large"
        @search="doSearch"
      />
    </div>


    <!-- 分类 + 标签 -->
    <a-tabs v-model:activeKey="selectedCategory" @change="doSearch">
      <a-tab-pane key="all" tab="全部" />
      <a-tab-pane v-for="category in categoryList" :key="category" :tab="category" />
    </a-tabs>
    <div class="tag-bar" style="margin-bottom: 16px">
      <span style="margin-right: 8px">标签：</span>
      <a-space :size="[0, 8]" wrap>
        <a-checkable-tag
          v-for="(tag, index) in tagList"
          :key="tag"
          v-model:checked="selectedTagList[index]"
          @change="doSearch"
        >
          {{ tag }}
        </a-checkable-tag>
      </a-space>
    </div>

<!--    &lt;!&ndash; 图片列表 &ndash;&gt;-->
<!--    <PictureList :dataList="dataList" :loading="loading" />-->

    <PictureListWaterFull :dataList="dataList"
                          @click-item="doClickPicture"/>


    <template>
      <!-- 在瀑布流组件下方添加加载提示 -->
      <div v-if="loadingMore" class="load-more">
        <a-spin />
        <span>加载中...</span>
      </div>
      <div v-if="noMoreData" class="no-more">
        没有更多数据了
      </div>
    </template>
  </div>
</template>
<script setup lang="ts">
// 数据
import { computed, onMounted, onUnmounted, reactive, ref, watch } from 'vue'
import {
  listPictureTagCategoryUsingGet,
  listPictureVoByPageUsingPost,
} from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import { useRouter } from 'vue-router'
import PictureList from '@/components/PictureList.vue'
import PictureListWaterFull from '@/components/PictureListWaterFull.vue'

const dataList = ref([])
const total = ref(0)
const loading = ref(true)

const categoryList = ref<string[]>([])
const selectedCategory = ref<string>('all')
const tagList = ref<string[]>([])
const selectedTagList = ref<string[]>([])
// 新增响应式变量
const initialLoading = ref(true) // 首次加载状态
const router = useRouter()
// 跳转至图片详情
const doClickPicture = (picture) => {
  router.push({
    path: `/picture/${picture.id}`,
  })
}

// 获取标签和分类选项
const getTagCategoryOptions = async () => {
  const res = await listPictureTagCategoryUsingGet()
  if (res.data.code === 0 && res.data.data) {
    // 转换成下拉选项组件接受的格式
    categoryList.value = res.data.data.categoryList ?? []
    tagList.value = res.data.data.tagList ?? []
  } else {
    message.error('加载分类标签失败，' + res.data.message)
  }
}

onMounted(() => {
  getTagCategoryOptions()
})

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({
  current: 1,
  pageSize: 12,
  sortField: 'createTime',
  sortOrder: 'descend',
})
// 修改 doSearch 方法
const doSearch = () => {
  searchParams.current = 1
  noMoreData.value = false // 重置加载状态
  fetchData()
}

// 监听分类变化
watch(selectedCategory, () => {
  searchParams.current = 1
  noMoreData.value = false
  fetchData()
})

// 新增响应式变量
const loadingMore = ref(false)
const noMoreData = ref(false)

// 修改后的 fetchData
const fetchData = async (append = false) => {
  if (!append) initialLoading.value = true
  if (!append) loading.value = true

  else loadingMore.value = true

  try {
    const params = {
      ...searchParams,
      tags: [],
    }
    if (selectedCategory.value !== 'all') {
      params.category = selectedCategory.value
    }
    selectedTagList.value.forEach((useTag, index) => {
      if (useTag) {
        params.tags.push(tagList.value[index])
      }
    })

    const res = await listPictureVoByPageUsingPost(params)

    if (res.data.code === 0) {
      const newData = res.data.data.records ?? []
      dataList.value = append
        ? [...dataList.value, ...newData]
        : newData

      noMoreData.value = newData.length < searchParams.pageSize
      if (append) searchParams.current += 1
    }
  } finally {
    if (!append) initialLoading.value = false
    loading.value = false
    loadingMore.value = false
  }
}

// 新增加载更多触发方法
const loadMore = () => {
  if (loadingMore.value || noMoreData.value) return
  searchParams.current += 1
  fetchData(true)
}
// 分页参数
const onPageChange = (page, pageSize) => {
  searchParams.current = page
  searchParams.pageSize = pageSize
  fetchData()
}

// 监听滚动事件
const scrollHandler = () => {
  const scrollTop = document.documentElement.scrollTop
  const windowHeight = window.innerHeight
  const documentHeight = document.documentElement.scrollHeight

  // 当滚动到底部 200px 范围内时触发
  if (scrollTop + windowHeight >= documentHeight - 200) {
    loadMore()
  }
}

// 在 mounted 钩子中强制加载
onMounted(() => {
  if (dataList.value.length === 0) {
    fetchData()
  }
  window.addEventListener('scroll', scrollHandler)
})

onUnmounted(() => {
  window.removeEventListener('scroll', scrollHandler)
})
</script>
<style scoped>
#homePage .search-bar {
  max-width: 480px;
  margin: 0 auto 16px;
}
.load-more {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  gap: 8px;
}

.no-more {
  text-align: center;
  color: #999;
  padding: 20px;
}

/* 防止快速滚动时的布局抖动 */
#homePage {
  min-height: 100vh;
  padding-bottom: 100px;
}
.full-loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
  padding: 40px;
  text-align: center;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px;
  color: #666;
}

.empty-state img {
  width: 200px;
  margin-bottom: 16px;
}
</style>
