<script setup>
import {
  Management,
  MessageBox,
  UserFilled,
  User,
  Crop,
  EditPen,
  SwitchButton,
  CaretBottom
} from '@element-plus/icons-vue'
import avatar from '@/assets/default.png'
import { useUserStore } from '@/stores'
import { onMounted } from 'vue'
// import { ElMessageBox, ElMessage } from 'element-plus'
import router from '@/router'
const userStore = useUserStore()
onMounted(() => {
  userStore.getUser()
})

const onCommand = (command) => {
  if (command === 'logout') {
    ElMessageBox.confirm('你确定要退出吗？', '温馨提示', {
      type: 'warning',
      confirmButtonText: '确定',
      cancelButtonText: '取消'
    }).then(() => {
      ElMessage({
        type: 'success',
        message: '退出成功'
      })
      userStore.removeToken()
      userStore.setUser({})
      router.push('/login')
    })
  } else {
    router.push(`/user/${command}`)
  }
}

const toHome = () => {
  router.push('/article/manage')
}
</script>

<template>
  <el-container class="layout-container">
    <el-aside width="200px">
      <div class="el-aside__logo" @click="toHome">
        <div class="icoimg"></div>
        <div class="icotext"><span>文章后台管理系统</span></div>
      </div>
      <el-menu
        active-text-color="#ffd04b"
        background-color="#014a60"
        :default-active="$route.path"
        text-color="#fff"
        router
      >
        <el-menu-item index="/article/channel">
          <el-icon><Management /></el-icon>
          <span>文章分类</span>
        </el-menu-item>
        <el-menu-item index="/article/manage">
          <el-icon><MessageBox /></el-icon>
          <span>文章管理</span>
        </el-menu-item>
        <el-sub-menu index="/user">
          <template #title>
            <el-icon><UserFilled /></el-icon>
            <span>个人中心</span>
          </template>
          <el-menu-item index="/user/profile">
            <el-icon><User /></el-icon>
            <span>基本资料</span>
          </el-menu-item>
          <el-menu-item index="/user/avatar">
            <el-icon><Crop /></el-icon>
            <span>更换头像</span>
          </el-menu-item>
          <el-menu-item index="/user/password">
            <el-icon><EditPen /></el-icon>
            <span>重置密码</span>
          </el-menu-item>
        </el-sub-menu>
      </el-menu>
    </el-aside>
    <el-container>
      <el-header>
        <div>
          <strong
            >文章作者：{{
              userStore.user.nickname || userStore.user.username
            }}</strong
          >
        </div>
        <el-dropdown placement="bottom-end" @command="onCommand">
          <span class="el-dropdown__box">
            <el-avatar
              shape="square"
              :src="userStore.user.user_pic || avatar"
            />
            <el-icon><CaretBottom /></el-icon>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item command="profile" :icon="User"
                >基本资料</el-dropdown-item
              >
              <el-dropdown-item command="avatar" :icon="Crop"
                >更换头像</el-dropdown-item
              >
              <el-dropdown-item command="password" :icon="EditPen"
                >重置密码</el-dropdown-item
              >
              <el-dropdown-item command="logout" :icon="SwitchButton"
                >退出登录</el-dropdown-item
              >
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </el-header>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<style lang="scss" scoped>
.layout-container {
  height: 100vh;
  .el-aside {
    background-color: #014a60;
    &__logo {
      display: flex;
      height: 65px;
      padding: 6%;
      background-color: rgb(209, 209, 209, 0.7);
      border-radius: 15px;
      margin: 5%;
      .icoimg {
        width: 50%;
        background: url('@/assets/icon.ico') no-repeat center / 50px auto;
      }
      .icotext {
        height: 65px;
        width: 50%;
        font-size: 17px;
        font-weight: bold;
        display: flex;
        justify-content: center; /* 水平居中 */
        align-items: center; /* 垂直居中 */
      }
    }
    .el-menu {
      border-right: none;
    }
  }
  .el-header {
    background-color: #fff;
    display: flex;
    align-items: center;
    justify-content: space-between;
    .el-dropdown__box {
      display: flex;
      align-items: center;
      .el-icon {
        color: #999;
        margin-left: 10px;
      }

      &:active,
      &:focus {
        outline: none;
      }
    }
  }
  .el-footer {
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 14px;
    color: #666;
  }
}
</style>
