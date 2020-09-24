<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img
          src="../assets/logo.png"
          alt=""
        >
      </div>
      <!-- 登录表单区域 -->
      <el-form
        ref="loginFormRef"
        label-width="0px"
        :rules="loginFormRules"
        class="login_form"
        :model="loginForm"
      >
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input
            prefix-icon="el-icon-user"
            v-model="loginForm.username"
          ></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            type="password"
            prefix-icon="el-icon-lock"
            v-model="loginForm.password"
          ></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
          <el-button
            type="info"
            @click="resetLoginForm"
          >重置</el-button>
          <el-button
            type="primary"
            @click="login"
          >登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      //登录表单的数据
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        //验证用户名是否合法
        username: [
          { required: true, message: '请输入登录用户名', trigger: 'blur' },
          { min: 1, max: 10, message: '长度在 1 到 10 个字符', trigger: 'blur' }
        ],
        //密码是否正确
        password: [
          { required: true, message: '请输入登录密码', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLoginForm() {
      this.$refs.loginFormRef.resetFields();
    },
    login() {
      this.$refs.loginFormRef.validate(async valid => {
        //false就不发请求
        if (!valid) return;
        const { data: res } = await this.$http.post('login', this.loginForm);
        console.log(res);
        if (res.meta.status !== 200) return this.$message.error('登录失败！')
        this.$message.success('登录成功')
        //1、将登录成功之后的 token 保存到客户端的sessionStorage中
        //1.1 项目中出了登录之外的其他API接口，必须在登录之后才能访问
        //1.2token 只应在当前网站打开期间生效，所以将token保存在sessionStorage中
        window.sessionStorage.setItem('token', res.data.token);
        //2、通过编程式导航跳转到后台祖业，路由地址是 /home
        this.$router.push('/home')
        //路由导航守卫

      })
    }
  },
}
</script>

<style lang="less" >
.login_container {
  height: 100%;
  background-color: #BBE6D6;
  position: relative;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: white;
  border-radius: 3px;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    box-shadow: 0 0 8px #ddd;
    border-radius: 50%;
    padding: 10px;
    position: absolute;
    background-color: white;
    left: 50%;
    transform: translate(-50%, -50%);
    img {
      height: 100%;
      width: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}
.login_form {
  position: absolute;
  padding: 0 20px;
  bottom: 0;
  width: 100%;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
