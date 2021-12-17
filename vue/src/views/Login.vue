<template>
  <div style="width: 100%; height: 100vh; background-color: darkslateblue; overflow: hidden">
    <div style="width: 400px; margin: 150px auto">
      <div style="color: #cccccc; font-size: 30px; text-align: center; padding: 30px 0">欢迎登录</div>
        <el-form ref="form" :model="form" size="normal" :rules="rules">
          <el-form-item prop="username">
            <el-input v-model="form.username" placeholder="请输入账号"></el-input>
          </el-form-item>
          <el-form-item  prop="password">
            <el-input v-model="form.password" show-password placeholder="请输入密码"></el-input>
          </el-form-item>
          <el-form-item>
            <el-radio v-model="form.role" :label="1">管理员</el-radio>
            <el-radio v-model="form.role" :label="2">普通用户</el-radio>
          </el-form-item>
          <el-form-item >
            <el-button style="width: 100%" type="primary" @click="login" >登录</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="text" @click="$router.push('/register')">注册 >> </el-button>
          </el-form-item>
        </el-form>
    </div>
  </div>
</template>

<script>
import {ElMessage} from "element-plus";
import request from "../utils/request";

export default {
  name: "Login",
  data() {
    return {
      form: {role: 1},
      rules: {
        username: [
          {required: true, message: '请输入用户名', trigger: 'blur'},
        ],
        password: [
          {required: true, message: '请输入密码', trigger: 'blur'},
        ],
      },
    }
  },
  created() {
    sessionStorage.removeItem("user")
  },
  methods: {
    login(){
      this.$refs['form'].validate((valid) => {
        if (valid) {
          request.post("/user/login", this.form).then(res =>{
            if(res.code ==='0'){
              ElMessage.success("登录成功")
              sessionStorage.setItem("user", JSON.stringify(res.data))//缓存用户信息
              this.$router.push("/")//登录成功后进行页面跳转，跳转到主页
            } else {
              ElMessage.error("用户名或者密码错误")
            }
          })
        }
        })
    }
  }
}
</script>

<style scoped>

</style>