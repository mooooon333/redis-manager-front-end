<template>
  <!--background-color: #f0f2f5; background-image: linear-gradient(0deg,#04c0c6,#2ba3de 51%,#835be3); -->
  <el-container style="background-image: linear-gradient(0deg,#1ac5fa,#2ba3de 51%,#1d71f2);">
    <!-- #1d71f2 #1ac5fa -->
    <img
      src="../assets/background.jpg"
      style="position: absolute; z-index: 0;left: 0; top:0; width: 100%; opacity:0.1;"
    />
    <div class="login-wrapper">
      <el-card shadow="hover" class="content-wrapper">
        <!-- <div slot="header">
          <b>Sign In</b>
        </div>-->
        <el-tabs v-model="activeName" @tab-click="handleClick">
          <el-tab-pane label="登录" name="first">
            <div>
              <el-form :model="user" :rules="rules" ref="user">
                <el-form-item prop="userName">
                  <el-input
                    prefix-icon="el-icon-user"
                    v-model.trim="user.userName"
                    placeholder="User Name"
                  ></el-input>
                </el-form-item>
                <el-form-item prop="password">
                  <el-input
                    prefix-icon="el-icon-key"
                    type="password"
                    v-model.trim="user.password"
                    placeholder="Password"
                    show-password
                  ></el-input>
                </el-form-item>
                <el-form-item>
                  <el-button type="primary" style="width: 100%;" @click="signIn('user')">Sign In</el-button>
                </el-form-item>
                <el-form-item v-if="authorization.enabled">
                  <el-button type="success" style="width: 100%;">
                    <el-link
                      :href="authorization.server + authorization.siteKey"
                      :underline="false"
                      style="color: #fff"
                    >{{ authorization.companyName }} Sign In</el-link>
                  </el-button>
                </el-form-item>
              </el-form>
            </div>
          </el-tab-pane>
          <el-tab-pane label="注册" name="second">
            <Register></Register>
          </el-tab-pane>
        </el-tabs>
      </el-card>
    </div>
  </el-container>
</template>

<script>
import API from '@/api/api.js'
import { store } from '@/vuex/store.js'
import message from '@/utils/message.js'
import Register from '@/components/Register'
import axios from 'axios'
export default {
  data () {
    return {
      user: {},
      authorization: {},
      rules: {
        userName: [
          { required: true, message: 'Please enter user name', trigger: 'blur' }
        ],
        password: [
          { required: true, message: 'Please enter password', trigger: 'blur' }
        ]
      },
      activeName: 'first'
    }
  },
  methods: {
    handleClick (tab, event) {},
    getAuthorization () {
      let url = '/system/getAuthorization'
      API.get(
        url,
        null,
        response => {
          let authorization = response.data.data
          authorization.server += authorization.server.endsWith('/') ? '' : '/'
          this.authorization = authorization
        },
        err => {
          message.error('Get authorization failed')
        }
      )
    },
    signIn (user) {
      this.$refs[user].validate(valid => {
        if (valid) {
          // axios.defaults.baseURL = '/apis2'
          let url = '/user/login'
          API.post(
            url,
            this.user,
            response => {
              let result = response.data
              // axios.defaults.baseURL = '/apis'
              if (result.code == 0) {
                store.dispatch('setUser', result.data)
                this.$router.push({ name: 'index' })
              } else {
                message.error('User name or password wrong')
              }
            },
            err => {
              // axios.defaults.baseURL = '/apis'
              message.error(err)
            }
          )
        }
      })
    }

    // getUserFromSession() {
    //   let url = "/user/getUserFromSession";
    //   API.get(
    //     url,
    //     null,
    //     response => {
    //       let result = response.data;
    //       if (result.code == 0) {
    //         store.dispatch("setUser", result.data);
    //         this.$router.push({ name: "index" });
    //       }
    //     },
    //     err => {
    //       message.error("Auto get user failed.");
    //     }
    //   );
    // }
  },
  mounted () {
    // this.getUserFromSession();
    this.getAuthorization()
    store.dispatch('setUser', {
      userRole: 2
    })
    store.dispatch('setCurrentGroup', {})
    store.dispatch('setGroupList', [])
  },
  components: {
    Register
  }
}
</script>

<style scoped>
.el-container {
  height: 100%;
}
.login-wrapper {
  background-color: #ffffff !important;
  margin: 0px auto;
  display: flex;
  align-self: center;
  margin-top: -5%;
}
.content-wrapper {
  width: 350px;
}
.el-divider {
  margin: 5px 0;
}
</style>
