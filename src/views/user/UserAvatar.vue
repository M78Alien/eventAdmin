<script setup>
import { Plus, Select } from '@element-plus/icons-vue'
import { ref } from 'vue'
import { userGetInfoService, userUploadAvatarService } from '@/api/user.js'
import { useUserStore } from '@/stores'
import { ElMessage } from 'element-plus'

const imgUrl = ref()
const userStore = useUserStore()

const getUserAratar = async () => {
  const res = await userGetInfoService()
  // console.log(res.data.data.user_pic)
  imgUrl.value = res.data.data.user_pic
}

getUserAratar()

const onUploadFile = (uploadFile) => {
  // imgUrl.value = URL.createObjectURL(uploadFile.raw)
  const reader = new FileReader()
  reader.readAsDataURL(uploadFile.raw)
  reader.onload = () => {
    imgUrl.value = reader.result
  }
}

const onUpdateAvatar = async () => {
  await userUploadAvatarService(imgUrl.value)
  await userStore.getUser()
  ElMessage({ message: '成功更换头像', type: 'success' })
  // console.log(imgUrl.value)
}
</script>

<template>
  <page-contains title="更换头像">
    <el-upload
      class="avatar-uploader"
      :auto-upload="false"
      :show-file-list="false"
      :on-change="onUploadFile"
    >
      <img v-if="imgUrl" :src="imgUrl" class="avatar" />
      <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
    </el-upload>
    <br />
    <el-button type="primary" @click="onUpdateAvatar">
      <el-icon><Select /></el-icon>上传头像
    </el-button>
  </page-contains>
</template>

<style lang="scss" scoped>
.avatar-uploader {
  :deep() {
    .avatar {
      height: 275px;
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
</style>
