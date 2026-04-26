<template>
  <div class="container">
  <div class="title">
      <div class="back-home">
          <el-icon><Back /></el-icon>
          <span>返回首页</span>
      </div>
      <div class="title-text">
        <h2>登录您的账户</h2>
        <p>输入您的登录信息</p>
      </div>
  </div>
  <div class="form-container">
    <el-form
    ref="ruleFormRef"
    :model="formData"
    :rules="rules"
    label-position="top"
    >
      <el-form-item label="用户名或邮箱" prop="username">
        <el-input v-model="formData.username" size="large" placeholder="请输入用户名或邮箱" />
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="formData.password" size="large" placeholder="请输入密码" type="password" show-password />
      </el-form-item>
      <el-button class="btn" size="large" type="primary" @click="submitForm(ruleFormRef)">登录</el-button>
    </el-form>
    <div class="footer">
        <p>还没有账号？<router-link to="/auth/register">去注册</router-link></p>
    </div>
  </div>
  </div>
</template>
<script setup>
import { ref,reactive } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { login } from '@/api/admin'

const router = useRouter()

const ruleFormRef =ref()
const formData = reactive({
  username: '',
  password: ''
})

const rules = reactive({
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' }
  ]
})

const submitForm = async(formEl) => {
  if(!formEl) return
  await formEl.validate((valid,fields) => {
    if (valid) {
      login(formData).then(data =>{
        if(!data.token){
          return console.error('登陆失败')
        }
        localStorage.setItem('token',data.token)
        localStorage.setItem('userInfo',JSON.stringify(data.userInfo))
        ElMessage.success('登录成功')
        router.push('/back/dashboard')
      })
      
    }
  })
}
</script>

<style scoped lang="scss">
.container{
  width:384px;
  .title{
    .back-home{
      margin-bottom: 60px;
    }
    .title-text{
      text-align: center;
      h2{
        font-size: 36px;
        margin-bottom: 10px;
      }
      p{
        font-size: 18px;
        color: #6b7280;
      }
    }
  }
  .form-container{
    .footer{
      padding:30px;
      text-align: center;
    }
    .btn{
      margin-bottom: 40px;
      width: 100%;
    }
  }
}

</style>