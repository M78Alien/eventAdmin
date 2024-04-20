<script setup>
import { ref } from 'vue'
import { Plus } from '@element-plus/icons-vue'
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'
import {
  artPublishService,
  artGetDetailService,
  artEditService
} from '@/api/article.js'
import { ElMessage } from 'element-plus'
import { baseURL } from '@/utils/request'

const visibleDrawer = ref(false)
const defaultForm = {
  title: '',
  cate_id: '',
  cover_img: '',
  content: '',
  state: ''
}
const imgUrl = ref('')
const formModel = ref({ ...defaultForm })
const formRef = ref()
const editorRef = ref()

async function imageUrlToFile(imageUrl, fileName) {
  try {
    // 通过axios获取图片的二进制数据
    // eslint-disable-next-line no-undef
    const response = await axios.get(imageUrl, {
      responseType: 'arraybuffer'
    })

    // 将二进制数据转换为Blob对象
    const blob = new Blob([response.data])

    // 创建一个File对象
    const file = new File([blob], fileName, {
      type: 'image/jpeg' // 这里假设图片类型为jpeg，可以根据实际情况调整
    })

    return file
  } catch (error) {
    // 如果发生错误，则返回错误信息
    return Promise.reject('转换图片失败：' + error.message)
  }
}

const open = async (row) => {
  visibleDrawer.value = true
  if (row.id) {
    // formModel.value = { ...row }
    const res = await artGetDetailService(row.id)
    formModel.value = res.data.data
    imgUrl.value = baseURL + formModel.value.cover_img
    formModel.value.cover_img = await imageUrlToFile(
      imgUrl.value,
      formModel.value.cover_img
    )
  } else {
    formModel.value = { ...defaultForm }
    imgUrl.value = ''
    editorRef.value.setHTML('')
  }
}
defineExpose({
  open
})

const emit = defineEmits(['success'])

const onUploadFile = (uploadFile) => {
  imgUrl.value = URL.createObjectURL(uploadFile.raw)
  formModel.value.cover_img = uploadFile.raw
}

const onPublish = async (state) => {
  formModel.value.state = state
  const fd = new FormData()
  for (let key in formModel.value) {
    fd.append(key, formModel.value[key])
  }
  if (formModel.value.id) {
    await artEditService(fd)
    ElMessage.success('编辑成功')
    visibleDrawer.value = false
    emit('success', 'edit')
  } else {
    await artPublishService(fd)
    ElMessage.success('添加成功')
    visibleDrawer.value = false
    emit('success', 'add')
  }
}

const rules = {
  title: [{ required: true, message: '请输入文章标题', trigger: 'blur' }],
  cate_id: [{ required: true, message: '请选择分类', trigger: 'blur' }],
  cover_img: [{ required: true, message: '请选择文章封面', trigger: 'blur' }],
  content: [{ required: true, message: '请输入文章内容', trigger: 'blur' }]
}
</script>

<template>
  <el-drawer
    v-model="visibleDrawer"
    :title="formModel.id ? '编辑文章' : '添加文章'"
    direction="rtl"
    size="50%"
  >
    <el-form
      :model="formModel"
      ref="formRef"
      :rules="rules"
      label-width="100px"
    >
      <el-form-item label="文章标题" prop="title">
        <el-input v-model="formModel.title" placeholder="请输入标题"></el-input>
      </el-form-item>
      <el-form-item label="文章分类" prop="cate_id">
        <channel-select
          v-model="formModel.cate_id"
          width="100%"
        ></channel-select>
      </el-form-item>
      <el-form-item label="文章封面" prop="cover_img">
        <el-upload
          class="avatar-uploader"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="onUploadFile"
        >
          <img v-if="imgUrl" :src="imgUrl" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
        </el-upload>
      </el-form-item>
      <el-form-item label="文章内容" prop="content">
        <div class="editor">
          <quill-editor
            ref="editorRef"
            theme="snow"
            v-model:content="formModel.content"
            contentType="html"
          ></quill-editor>
        </div>
      </el-form-item>
      <el-form-item>
        <el-button @click="onPublish('已发布')" type="primary">发布</el-button>
        <el-button @click="onPublish('草稿')" type="info">草稿</el-button>
      </el-form-item>
    </el-form>
  </el-drawer>
</template>

<style lang="scss" scoped>
.avatar-uploader {
  :deep() {
    .avatar {
      width: 178px;
      height: 178px;
      display: block;
    }
    .el-upload {
      border: 1px dashed var(--el-border-color);
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hidden;
      transition: var(--el-transition-duration-fast);
    }
    .el-upload:hover {
      border-color: var(--el-color-primary);
    }
    .el-icon.avatar-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 178px;
      height: 178px;
      text-align: center;
    }
  }
}
.editor {
  width: 100%;
  :deep(.ql-editor) {
    min-height: 200px;
  }
}
</style>
