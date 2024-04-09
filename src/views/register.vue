<template>
  <div class="register">
    <el-form ref="registerRef" :model="registerForm" :rules="registerRules" class="register-form">
      <h3 class="title">OJ</h3>
      <el-form-item prop="username">
        <el-input
            v-model="registerForm.username"
            type="text"
            size="large"
            auto-complete="off"
            placeholder="Username"
        >
          <template #prefix>
            <svg-icon icon-class="user" class="el-input__icon input-icon"/>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="email">
        <el-input
            v-model="registerForm.email"
            type="text"
            size="large"
            auto-complete="off"
            placeholder="Email"
        >
          <template #prefix>
            <svg-icon icon-class="email" class="el-input__icon input-icon"/>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="gender">
        <el-select v-model="registerForm.gender" size="large">
          <template #prefix>
            <svg-icon icon-class="build" class="el-input__icon input-icon"/>
          </template>
          <el-option label="male" value="male"></el-option>
          <el-option label="female" value="female"></el-option>
          <el-option label="secret" value="secret"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="firstName">
        <el-input
            v-model="registerForm.firstName"
            type="text"
            size="large"
            auto-complete="off"
            placeholder="FirstName"
        >
          <template #prefix>
            <svg-icon icon-class="user" class="el-input__icon input-icon"/>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="lastName">
        <el-input
            v-model="registerForm.lastName"
            type="text"
            size="large"
            auto-complete="off"
            placeholder="LastName"
        >
          <template #prefix>
            <svg-icon icon-class="user" class="el-input__icon input-icon"/>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
            v-model="registerForm.password"
            type="password"
            size="large"
            auto-complete="off"
            placeholder="password"
            @keyup.enter="handleRegister"
        >
          <template #prefix>
            <svg-icon icon-class="password" class="el-input__icon input-icon"/>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="confirmPassword">
        <el-input
            v-model="registerForm.confirmPassword"
            type="password"
            size="large"
            auto-complete="off"
            placeholder="confirmPassword"
            @keyup.enter="handleRegister"
        >
          <template #prefix>
            <svg-icon icon-class="password" class="el-input__icon input-icon"/>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="code" v-if="captchaEnabled">
        <el-input
            size="large"
            v-model="registerForm.code"
            auto-complete="off"
            placeholder="code"
            style="width: 63%"
            @keyup.enter="handleRegister"
        >
          <template #prefix>
            <svg-icon icon-class="validCode" class="el-input__icon input-icon"/>
          </template>
        </el-input>
        <div class="register-code">
          <img :src="codeUrl" @click="getCode" class="register-code-img"/>
        </div>
      </el-form-item>
      <el-form-item style="width:100%;">
        <el-popconfirm
            title="确定发送注册验证邮件吗？"
            @confirm.prevent="handleRegister"
        >
          <template #reference>
            <el-button
                :loading="loading"
                size="large"
                type="primary"
                style="width:100%;">
              <span v-if="!loading">注 册</span>
              <span v-else>注 册 中...</span>
            </el-button>
          </template>
        </el-popconfirm>
        <div style="float: right;">
          <router-link class="link-type" :to="'/login'">使用已有账户登录</router-link>
        </div>
      </el-form-item>
    </el-form>
    <!--  底部  -->
    <div class="el-register-footer">
      <span>Copyright © 2018-2024 ruoyi.vip All Rights Reserved.</span>
    </div>
  </div>

  <div class="el-code-dialog">
    <el-dialog title='输入邮箱验证码'
               :close-on-click-modal="false"
               :close-on-press-escape="false"
               :show-close="false"
               center
               v-model="innerVisible"
               width="35%"
    >
      <template>

      </template>
      <template #footer>
        <el-button @click="innerVisible = false">返回</el-button>
      </template>
    </el-dialog>
  </div>

</template>

<script setup>
import {ElMessage, ElMessageBox} from "element-plus";
import {getCodeImg, register} from "@/api/login";



const router = useRouter();
const {proxy} = getCurrentInstance();

const registerForm = ref({
  username: "",
  email: "",
  firstName: "",
  lastName: "",
  gender: "",
  password: "",
  confirmPassword: "",
  code: "",
  uuid: ""
});

const innerVisible = ref(true);

const emailCode = ref({
  code: ""
});

const equalToPassword = (rule, value, callback) => {
  if (registerForm.value.password !== value) {
    callback(new Error("两次输入的密码不一致"));
  } else {
    callback();
  }
};

const registerRules = {
  username: [
    {required: true, trigger: "blur", message: "请输入您的账号"},
    {min: 6, max: 20, message: "用户账号长度必须介于 6 和 20 之间", trigger: "blur"}
  ],
  password: [
    {required: true, trigger: "blur", message: "请输入您的密码"},
    {min: 8, max: 20, message: "用户密码长度必须介于 8 和 20 之间", trigger: "blur"},
    {pattern: /^[^<>"'|\\]+$/, message: "不能包含非法字符：< > \" ' \\\ |", trigger: "blur"}
  ],
  confirmPassword: [
    {required: true, trigger: "blur", message: "请再次输入您的密码"},
    {required: true, validator: equalToPassword, trigger: "blur"}
  ],
  code: [{required: true, trigger: "change", message: "请输入验证码"}]
};

const codeUrl = ref("");
const loading = ref(false);
const captchaEnabled = ref(true);


function handleRegister() {
  proxy.$refs.registerRef.validate(valid => {
    if (valid) {
      loading.value = true;
      register(registerForm.value).then(res => {
        if (res.code < 0) {
          ElMessage({message: res.message, type: 'warning'});
          return;
        }
        console.log(res)
        const username = registerForm.value.username;
        ElMessageBox.alert("<font color='red'>验证邮件已发送，请查收！</font>", "系统提示", {
          dangerouslyUseHTMLString: true,
          type: "success",
        }).then(() => {
          innerVisible.value = true;
          // router.push("/verify");
        }).catch(() => {
        });
      }).catch(() => {
        loading.value = false;
        if (captchaEnabled) {
          getCode();
        }
      });
    }
  });
}

function getCode() {
  getCodeImg().then(res => {
    // captchaEnabled.value = res.captchaEnabled === undefined ? true : res.captchaEnabled;
    if (captchaEnabled.value) {
      codeUrl.value = "data:image/gif;base64," + res.data.img;
      ;
      registerForm.value.uuid = res.data.uuid;
    }
  });
}

getCode();
</script>

<style lang='scss' scoped>
@import "uview-plus/index.scss";
.register {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  background-image: url("../assets/images/login-background.jpg");
  background-size: cover;
}

.title {
  margin: 0px auto 30px auto;
  text-align: center;
  color: #707070;
}

.register-form {
  border-radius: 6px;
  background: #ffffff;
  width: 400px;
  padding: 25px 25px 5px 25px;

  .el-input {
    height: 40px;

    input {
      height: 40px;
    }
  }

  .el-select {
    height: 40px;
    width: 350px;

    .el-option {
      height: 40px;
    }
  }

  .input-icon {
    height: 39px;
    width: 14px;
    margin-left: 0px;
  }
}

.register-tip {
  font-size: 13px;
  text-align: center;
  color: #bfbfbf;
}

.register-code {
  width: 33%;
  height: 40px;
  float: right;

  img {
    cursor: pointer;
    vertical-align: middle;
  }
}

.el-register-footer {
  height: 40px;
  line-height: 40px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: #fff;
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
}

.register-code-img {
  height: 32px;
  padding-left: 12px;
}

</style>
