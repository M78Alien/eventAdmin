<script setup>
import { ref } from 'vue'
import { artGetDetailService } from '@/api/article.js'
import { formatTime } from '@/utils/format'
import { baseURL } from '@/utils/request'

const content = ref({
  title: '',
  cate_id: '',
  cover_img: '',
  content: '',
  state: ''
})
const visibleDrawer = ref(false)
const imgUrl = ref()

const open = async (row) => {
  visibleDrawer.value = true
  const res = await artGetDetailService(row.id)
  content.value = res.data.data
  imgUrl.value = baseURL + content.value.cover_img
  console.log(content.value)
}

defineExpose({
  open
})
</script>

<template>
  <el-drawer
    v-model="visibleDrawer"
    title="文章详情"
    direction="rtl"
    size="50%"
  >
    <div>
      <h2>{{ content.title }}</h2>
    </div>
    <div>
      <p>作者：{{ content.nickname }}</p>
      <p>文章分类：{{ content.cate_name }}</p>
      <p>发布时间：{{ formatTime(content.pub_date) }}</p>
      <hr />
    </div>
    <br />
    <div>
      <el-image style="width: 50%" :src="imgUrl"></el-image>
    </div>
    <div v-html="content.content"></div>
  </el-drawer>
</template>
