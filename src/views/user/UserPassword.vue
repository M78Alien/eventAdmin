<script setup>
import { ref } from 'vue'
import { userUpdatePwdService } from '@/api/user.js'
import useRouter from '@/router'
import { useUserStore } from '@/stores'

const pwdFrom = ref({
  old_pwd: '',
  new_pwd: '',
  re_pwd: ''
})
const formRef = ref()
const useStore = useUserStore()

const checkOldSame = (rule, value, callback) => {
  if (value === pwdFrom.value.old_pwd) {
    callback(new Error('原密码和新密码不能一样!'))
  } else {
    callback()
  }
}

const checkNewSame = (rule, value, callback) => {
  if (value !== pwdFrom.value.new_pwd) {
    callback(new Error('两次输入的密码不一致！'))
  } else {
    callback()
  }
}

const onSubmit = async () => {
  const valid = await formRef.value.validate()
  if (valid) {
    await userUpdatePwdService(pwdFrom.value)
    ElMessage({ message: '密码更换成功', type: 'success' })
    useStore.setToken('')
    useStore.setUser({})
    useRouter.push('/login')
    ElMessage({ message: '请重新登录', type: 'success' })
  }
}

const onReset = () => {
  formRef.value.resetFields()
}

const rules = {
  old_pwd: [
    { required: true, message: '请输入原密码', trigger: 'blur' },
    {
      pattern: /^\S{6,15}$/,
      message: '密码必须是6-15位的非空字符',
      trigger: 'blur'
    }
  ],
  new_pwd: [
    { required: true, message: '请输入新密码', trigger: 'blur' },
    {
      pattern: /^\S{6,15}$/,
      message: '密码必须是6-15位的非空字符',
      trigger: 'blur'
    },
    { validator: checkOldSame, trigger: 'blur' }
  ],
  re_pwd: [
    { required: true, message: '请再次输入新密码', trigger: 'blur' },
    {
      pattern: /^\S{6,15}$/,
      message: '密码必须是6-15位的非空字符',
      trigger: 'blur'
    },
    { validator: checkNewSame, trigger: 'blur' }
  ]
}
</script>

<template>
  <page-contains title="重置密码">
    <el-form
      :model="pwdFrom"
      :rules="rules"
      ref="formRef"
      label-width="100px"
      size="large"
      style="max-width: 600px"
    >
      <el-form-item label="原密码" prop="old_pwd">
        <el-input v-model="pwdFrom.old_pwd" type="password"></el-input>
      </el-form-item>
      <el-form-item label="新密码" prop="new_pwd">
        <el-input v-model="pwdFrom.new_pwd" type="password"></el-input>
      </el-form-item>
      <el-form-item label="确认新密码" prop="re_pwd">
        <el-input v-model="pwdFrom.re_pwd" type="password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">修改密码</el-button>
        <el-button @click="onReset">重置</el-button>
      </el-form-item>
    </el-form>
  </page-contains>
</template>
