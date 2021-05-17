<template>
  <view class="register">

    <view class="content">
      <!-- 头部logo -->
      <view class="header" style="background-color:#FFFFFF">
        <image :src="logoImage"></image>
      </view>
      <!-- 主体 -->
      <view class="main">
        <wInput v-model="phoneData" type="text" placeholder="邮箱"></wInput>
        <wInput v-model="passData" type="password" placeholder="登录密码" isShowPass></wInput>
      </view>

      <wButton class="wbutton" text="登 录" :rotate="isRotate" @click.native="startReg()"></wButton>

      <!-- 底部信息 -->
      <view class="footer">
        <text @tap="isShowAgree" class="cuIcon" :class="showAgree?'cuIcon-radiobox':'cuIcon-round'"> 同意</text>
        <!-- 协议地址 -->
        <navigator url="../login/userprotocol">《协议》</navigator>
      </view>
    </view>
  </view>
</template>

<script>
  var _this;
  import wInput from '@/components/watch-login/watch-input.vue';
  import wButton from '@/components/watch-login/watch-button.vue';
  import until from '@/components/utils/util.js';
  import http from '@/components/utils/http.js';
  export default {
    data() {
      return {
        //logo图片 base64
        logoImage: 'data:image/png;base64,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',
        phoneData: '', // 用户/电话
        passData: '', //密码
        showAgree: true, //协议是否选择
        isRotate: false, //是否加载旋转
      }
    },
    components: {
      wInput,
      wButton,
    },
    mounted() {
      _this = this;
    },
    methods: {
      isShowAgree() {
        //是否选择协议
        _this.showAgree = !_this.showAgree;
      },
      startReg() {
        //登录
        if (this.isRotate) {
          //判断是否加载中，避免重复点击请求
          return false;
        }
        if (this.showAgree == false) {
          uni.showToast({
            icon: 'none',
            position: 'bottom',
            title: '请先同意《协议》'
          });
          return false;
        }
        if (this.phoneData.length == '') {
          uni.showToast({
            icon: 'none',
            position: 'bottom',
            title: '邮箱不能为空'
          });
          return false;
        }
        if (!until.checkEmail(this.phoneData)) {
          uni.showToast({
            icon: 'none',
            position: 'bottom',
            title: '邮箱格式不正确'
          });
          return false;
        }
        if (this.passData.length == '') {
          uni.showToast({
            icon: 'none',
            position: 'bottom',
            title: '密码不正确'
          });
          return false;
        }
        let opts = {
          url: 'auth/login',
          method: 'post'
        };
        let param = {
          email: this.phoneData,
          password: this.passData,
          userOrAdmin: '1'
        };
        _this.isRotate = true
        http.httpTokenRequest(opts, param).then(res => {
          if (res.data.success === undefined) {
            uni.setStorage({
              key: 'Authorization',
              data: res.header.Authorization,
              success: function() {
                _this.isRotate = false
                uni.setStorage({
                  key: 'islogin',
                  data: res.data,
                  success: function() {
                    uni.reLaunch({
                      url: '../main/main',
                    });
                  }
                });
              }
            });
          } else {
            _this.isRotate = false
            uni.showToast({
              icon: 'none',
              title: res.data.error,
              duration: 2000
            });
          }
          //打印请求返回的数据
        }, error => {
          console.log(error);
        })
      },
      showislogin() {
        var loginis = uni.getStorageSync('islogin');
        if (loginis !== '') {
          uni.reLaunch({
            url: '../main/main',
          });
        }
      },
    },
    onLoad() {
      let that = this;
      that.showislogin();
    }
  }
</script>

<style>
  @import url("../../components/login/icon.css");
  @import url("../../components/login/main.css");
</style>
