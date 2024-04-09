<template>
<!--  <div class="verify">-->
<!--    <el-form ref="loginRef" :model="emailCodeForm" :rules="codeRules" class="register-code-form">-->
<!--      <h3 class="title">OJ 邮箱验证码</h3>-->
<!--      <el-form-item prop="code">-->
<!--        <el-input-->
<!--            v-model="emailCodeForm.code"-->
<!--            size="large"-->
<!--            auto-complete="off"-->
<!--            placeholder="邮箱验证码"-->
<!--            style="width: 100%"-->
<!--            @keyup.enter="handleVerifyCode"-->
<!--        >-->
<!--          <template #prefix>-->
<!--            <svg-icon icon-class="validCode" class="el-input__icon input-icon"/>-->
<!--          </template>-->
<!--        </el-input>-->
<!--      </el-form-item>-->
<!--    </el-form>-->

<!--  </div>-->

  <div class="verify">
    <el-button type="primary" @click="dialogVisible = true">点击打开 Dialog</el-button>

    <el-dialog
        title="提示"
        v-model="dialogVisible"
        width="800px"
        :before-close="handleClose"
        append-to-body
    >
      <span>这是一段信息</span>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
    </el-dialog>

  </div>

</template>


<script setup>

import {verifyEmailCode} from "@/api/login.js";
import {ElMessageBox} from "element-plus";

// const router = useRouter();
const { proxy } = getCurrentInstance();

const innerVisible = ref(true);

const codeRules = {
  code: [{required: true, trigger: "change", message: "请输入邮箱里的验证码"}]
};

const emailCodeForm = ref({
  code: ""
});

function handleVerifyCode() {
  verifyEmailCode().then(res => {
    ElMessageBox.alert("<font color='red'>注册成功！</font>", "系统提示", {
      dangerouslyUseHTMLString: true,
      type: "success"
    }).then(() => {
      router.push("/login");
    })
  })
}
</script>


<style scoped lang="scss">
.verify {
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

.register-code-form {
  border-radius: 6px;
  background: #ffffff;
  width: 400px;
  padding: 25px 25px 5px 25px;

  .el-input {
    height: 40px;

    input {
      width: 100px;
    }
  }
}
</style>