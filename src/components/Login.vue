<template>
    <div class="login">
        <div class="login_box">
            <!-- 头像区域 -->
            <div class="avatar_box">
                <img src="../assets/logo.png" alt="">
            </div>
            <!-- 表单区域 -->
            <el-form class="loginForm" :model="loginFormInfo" :rules="loginFormRules" ref="ruleForm">
                <el-form-item prop="username">
                    <el-input prefix-icon="el-icon-user" v-model="loginFormInfo.username"></el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input prefix-icon="el-icon-lock" v-model="loginFormInfo.password"></el-input>
                </el-form-item>
                <el-form-item class="btns">
                    <el-button type="primary" @click="login">登录</el-button>
                    <el-button type="info" @click="resetForm">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>
export default {
  data () {
    return {
      loginFormInfo: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 20, message: '长度在 6 到 20 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetForm () {
      this.$refs.ruleForm.resetFields()
    },
    login () {
      this.$refs.ruleForm.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginFormInfo)
        if (res.meta.status !== 200) return this.$message.error('登录失败！')
        this.$message.success('登录成功')
        // 1. 将 token 保存到 sessionStorage 中
        // 2. 跳转路由
        window.sessionStorage.setItem('token', res.data.token)
        // 编程式路由
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login {
    position: relative;
    height: 100%;
    background-color: #2b4b6b;
}

.login_box {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 450px;
    height: 300px;
    border-radius: 3px;
    background-color: #fff;

    .avatar_box {
        position: absolute;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 130px;
        height: 130px;
        padding: 10px;
        border: 1px solid #eee;
        border-radius: 50%;
        background-color: #fff;
        box-shadow: 0 0 10px #ddd;
        img {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background-color: #eee;
        }
    }

    .loginForm {
        position: absolute;
        bottom: 0;
        width: 100%;
        padding: 0 20px;
        box-sizing: border-box;

        .btns {
            display: flex;
            justify-content: flex-end;
        }
    }
}
</style>
