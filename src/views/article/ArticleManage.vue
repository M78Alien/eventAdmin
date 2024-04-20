<script setup>
import { Delete, Edit } from '@element-plus/icons-vue'
import { ref } from 'vue'
import { artGetListService, artDelService } from '@/api/article.js'
import { formatTime } from '@/utils/format'
import { ElMessage, ElMessageBox } from 'element-plus'

const articleList = ref([])
const total = ref(0)
const params = ref({
  pagenum: 1,
  pagesize: 5,
  cate_id: '',
  state: ''
})
const loading = ref(false)
const articleEditRef = ref()
const articleContentRef = ref()

const onAddArticle = () => {
  articleEditRef.value.open({})
}
const onEditArticle = (row) => {
  articleEditRef.value.open(row)
}
const onDeleteArticel = async (row) => {
  await ElMessageBox.confirm('你确定要删除该文章吗？', '温馨提示', {
    type: 'warning',
    confirmButtonText: '确认',
    cancelButtonText: '取消'
  })
  await artDelService(row.id)
  ElMessage({ type: 'success', message: '删除成功' })
  getArticalList()
}

const getArticalList = async () => {
  loading.value = true
  const res = await artGetListService(params.value)
  articleList.value = res.data.data
  total.value = res.data.total
  loading.value = false
}
getArticalList()

const onSizeChange = (size) => {
  params.value.pagenum = 1
  params.value.pagesize = size
  getArticalList()
}
const onCurrentChange = (page) => {
  params.value.pagenum = page
  getArticalList()
}

const onSearch = () => {
  params.value.pagenum = 1
  getArticalList()
}
const onReset = () => {
  params.value.pagenum = 1
  params.value.cate_id = ''
  params.value.state = ''
  getArticalList()
}

const onSuccess = (type) => {
  if (type === 'add') {
    const lastPage = Math.ceil((total.value + 1) / params.value.pagesize)
    params.value.pagenum = lastPage
  }
  getArticalList()
}

const onArticleCont = (row) => {
  // console.log(row)
  articleContentRef.value.open(row)
}
</script>

<template>
  <page-contains title="文章管理">
    <template #extra>
      <el-button type="primary" @click="onAddArticle">发布文章</el-button>
    </template>
    <el-form inline>
      <el-form-item label="文章分类：">
        <channel-select v-model="params.cate_id"></channel-select>
      </el-form-item>
      <el-form-item label="发布状态：">
        <el-select v-model="params.state" style="width: 200px">
          <el-option label="已发布" value="已发布"></el-option>
          <el-option label="草稿" value="草稿"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSearch">搜索</el-button>
        <el-button @click="onReset">重置</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="articleList"
      v-loading="loading"
      style="width: 100%"
      stripe
    >
      <el-table-column label="文章标题" width="400">
        <template #default="{ row }">
          <el-link
            @click="onArticleCont(row)"
            type="primary"
            :underline="false"
            >{{ row.title }}</el-link
          >
        </template>
      </el-table-column>
      <el-table-column label="分类" prop="cate_name"></el-table-column>
      <el-table-column label="发表时间">
        <template #default="{ row }">
          {{ formatTime(row.pub_date) }}
        </template>
      </el-table-column>
      <el-table-column label="状态" prop="state"></el-table-column>
      <el-table-column label="操作" width="150">
        <template #default="{ row }">
          <el-button
            :icon="Edit"
            round
            plain
            type="primary"
            @click="onEditArticle(row)"
          ></el-button>
          <el-button
            :icon="Delete"
            round
            plain
            type="danger"
            @click="onDeleteArticel(row)"
          ></el-button>
        </template>
      </el-table-column>
      <template #empty>
        <el-empty description="没有数据"></el-empty>
      </template>
    </el-table>
    <el-pagination
      v-model:current-page="params.pagenum"
      v-model:page-size="params.pagesize"
      :page-sizes="[3, 5, 10]"
      layout="jumper, total, sizes, prev, pager,
      next"
      background
      :total="total"
      @size-change="onSizeChange"
      @current-change="onCurrentChange"
      style="margin-top: 20px; justify-content: flex-end"
    ></el-pagination>
  </page-contains>
  <article-edit ref="articleEditRef" @success="onSuccess"></article-edit>
  <article-content ref="articleContentRef"></article-content>
</template>
