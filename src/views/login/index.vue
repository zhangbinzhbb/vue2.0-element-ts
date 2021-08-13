
<template>
  <div class="login-container">
    <div class="login-layout">
      <div class="login-head">
        <img
          class="logo"
          src="https://s.weituibao.com/1582958061265/mlogo.png"
        />
        <div class="name">
          <div class="title">乐摇摇</div>
          <div class="tips">
            {{ $t("login.title") }} <lang-select class="set-language" />
          </div>
        </div>
      </div>
      <el-form
        ref="loginForm"
        :model="loginForm"
        :rules="loginRules"
        class="login-form"
        autocomplete="on"
        label-position="left"
      >
        <el-form-item prop="username">
          <span class="svg-container">
            <svg-icon name="user" />
          </span>
          <el-input
            ref="username"
            v-model="loginForm.username"
            :placeholder="$t('login.username')"
            name="username"
            type="text"
            tabindex="1"
            autocomplete="on"
          />
        </el-form-item>

        <el-tooltip
          v-model="capsTooltip"
          content="Caps lock is On"
          placement="right"
          manual
        >
          <el-form-item prop="password">
            <span class="svg-container">
              <svg-icon name="password" />
            </span>
            <el-input
              :key="passwordType"
              ref="password"
              v-model="loginForm.password"
              :type="passwordType"
              :placeholder="$t('login.password')"
              name="password"
              tabindex="2"
              autocomplete="on"
              @keyup.native="checkCapslock"
              @blur="capsTooltip = false"
              @keyup.enter.native="handleLogin"
            />
            <span class="show-pwd" @click="showPwd">
              <svg-icon
                :name="passwordType === 'password' ? 'eye-off' : 'eye-on'"
              />
            </span>
          </el-form-item>
        </el-tooltip>

        <el-button
          :loading="loading"
          type="primary"
          style="width: 100%; margin-bottom: 30px"
          @click.native.prevent="handleLogin"
        >
          {{ $t("login.logIn") }}
        </el-button>

        <div style="position: relative">
          <div class="tips">
            <span>{{ $t("login.username") }} : admin </span>
            <span>{{ $t("login.password") }} : {{ $t("login.any") }} </span>
          </div>
          <div class="tips">
            <span>{{ $t("login.username") }} : editor </span>
            <span>{{ $t("login.password") }} : {{ $t("login.any") }} </span>
          </div>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import { Route } from "vue-router";
import { Dictionary } from "vue-router/types/router";
import { Form as ElForm, Input } from "element-ui";
import { UserModule } from "@/store/modules/user";
import { isValidUsername } from "@/utils/validate";
import LangSelect from "@/components/LangSelect/index.vue";

@Component({
  name: "Login",
  components: {
    LangSelect,
  },
})
export default class extends Vue {
  private validateUsername = (rule: any, value: string, callback: Function) => {
    if (!isValidUsername(value)) {
      callback(new Error("Please enter the correct user name"));
    } else {
      callback();
    }
  };

  private validatePassword = (rule: any, value: string, callback: Function) => {
    if (value.length < 6) {
      callback(new Error("The password can not be less than 6 digits"));
    } else {
      callback();
    }
  };

  private loginForm = {
    username: "admin",
    password: "111111",
  };

  private loginRules = {
    username: [{ validator: this.validateUsername, trigger: "blur" }],
    password: [{ validator: this.validatePassword, trigger: "blur" }],
  };

  private passwordType = "password";
  private loading = false;
  private showDialog = false;
  private capsTooltip = false;
  private redirect?: string;
  private otherQuery: Dictionary<string> = {};

  @Watch("$route", { immediate: true })
  private onRouteChange(route: Route) {
    // TODO: remove the "as Dictionary<string>" hack after v4 release for vue-router
    // See https://github.com/vuejs/vue-router/pull/2050 for details
    const query = route.query as Dictionary<string>;
    if (query) {
      this.redirect = query.redirect;
      this.otherQuery = this.getOtherQuery(query);
    }
  }

  mounted() {
    if (this.loginForm.username === "") {
      (this.$refs.username as Input).focus();
    } else if (this.loginForm.password === "") {
      (this.$refs.password as Input).focus();
    }
  }

  private checkCapslock(e: KeyboardEvent) {
    const { key } = e;
    this.capsTooltip =
      key !== null && key.length === 1 && key >= "A" && key <= "Z";
  }

  private showPwd() {
    if (this.passwordType === "password") {
      this.passwordType = "";
    } else {
      this.passwordType = "password";
    }
    this.$nextTick(() => {
      (this.$refs.password as Input).focus();
    });
  }

  private handleLogin() {
    (this.$refs.loginForm as ElForm).validate(async (valid: boolean) => {
      if (valid) {
        this.loading = true;
        await UserModule.Login(this.loginForm);
        this.$router
          .push({
            path: this.redirect || "/",
            query: this.otherQuery,
          })
          .catch((err) => {
            console.warn(err);
          });
        // Just to simulate the time of the request
        setTimeout(() => {
          this.loading = false;
        }, 0.5 * 1000);
      } else {
        return false;
      }
    });
  }

  private getOtherQuery(query: Dictionary<string>) {
    return Object.keys(query).reduce((acc, cur) => {
      if (cur !== "redirect") {
        acc[cur] = query[cur];
      }
      return acc;
    }, {} as Dictionary<string>);
  }
}
</script>

<style lang="scss">
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;
    input {
      height: 47px;
      border: 0px;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      -webkit-appearance: none;
      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $loginBg inset !important;
        -webkit-text-fill-color: #000 !important;
      }
    }
  }
  .el-form-item {
    border-radius: 5px;
    color: #454545;
    border: 1px solid #ccc;
  }
}
</style>

<style lang="scss" scoped>
.login-container {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  overflow: hidden;
  background-color: $loginBg;
  .login-layout {
    width: 420px;
    height: 500px;
    background-color: #fff;
    border-radius: 4px;
    box-shadow: 0 21px 41px 0 rgba(0, 0, 0, 0.2);
    .login-head {
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 40px 0 20px 0;
      img {
        width: 100px;
        height: 100px;
        margin-right: 20px;
      }
      .title {
        font-size: 28px;
        color: #1baeae;
        font-weight: bold;
      }
      .tips {
        font-size: 12px;
        color: #999;
      }
    }
  }
  .login-form {
    width: 70%;
    margin: 0 auto;
  }
  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $darkGray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }
  .show-pwd {
    position: absolute;
    right: 10px;
    top: 6px;
    font-size: 16px;
    color: $darkGray;
    cursor: pointer;
    user-select: none;
  }
}
</style>
