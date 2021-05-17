<template>
  <view>
    <cu-custom bgColor="bg-gradual-blue" :isBack="true" v-if="istowork">
      <block slot="backText">返回</block>
      <block slot="content">{{ workinfodata.order_num }}工单</block>
    </cu-custom>
    <view class="cu-bar bg-white bg-gradual-blue" v-if="istoworkrm">
      <view class="action" @click="rmtoworklist">
        <text class="cuIcon-back text-gray"></text> 返回
      </view>
      <view class="content text-bold">
        {{ workinfodata.order_num }}工单
      </view>
    </view>
    <view class="cu-bar bg-white solid-bottom">
      <view class="action">
        <text class="cuIcon-title text-orange"></text> 工单步骤
      </view>
    </view>
    <view class="bg-white padding">
      <view class="cu-steps">
        <view class="cu-item" :class="index>num?'':'text-blue'" v-for="(item,index) in numList" :key="index">
          <text class="num" :data-index="index + 1"></text> {{item.name}}
        </view>
      </view>
    </view>
    <!-- 开始接单工单 -->
    <view class="shadow-blur bg-white margin-top" v-if="num == 0">
      <view class="flex  p-xs  mb-sm">
        <view class="flex-sub bg-grey padding-sm margin-xs radius">工单编号:{{ workinfodata.order_num }}</view>
        <!-- <view class="flex-twice bg-grey padding-sm margin-xs radius">2</view>
        <view class="flex-treble bg-grey padding-sm margin-xs radius">3</view> -->
      </view>
      <view class="flex  p-xs mb-sm">
        <view class="flex-sub bg-grey padding-sm margin-xs radius">设备编号:{{ workinfodata.devicenumber }}</view>
      </view>
      <view class="flex  p-xs mb-sm">
        <view class="flex-sub bg-grey padding-sm margin-xs radius">收单人:{{ workinfodata.name}}</view>
        <view class="flex-twice bg-grey padding-sm margin-xs radius">工单时间:{{ workinfodata.created_at }}</view>
      </view>
      <view class="flex  p-xs mb-sm">
        <view class="flex-sub bg-grey padding-sm margin-xs radius">设备类型:{{ workinfodata.tname}}</view>
        <view class="flex-sub bg-grey padding-sm margin-xs radius">设备状态:{{ workinfodata.status_name}}</view>
      </view>
      <view class="flex  p-xs mb-sm">
        <view class="flex-sub bg-grey padding-sm margin-xs radius">报警开始时间:{{ workinfodata.start_time}}</view>
      </view>
      <view class="flex  p-xs mb-sm">
        <view class="flex-sub bg-grey padding-sm margin-xs radius">设备介质:{{ workinfodata.gas}}</view>
        <view class="flex-sub bg-grey padding-sm margin-xs radius">设备浓度:{{ workinfodata.concentration }}</view>
        <!-- <view class="flex-sub bg-grey padding-sm margin-xs radius">设备单位:{{ workinfodata.danwei }}</view> -->
      </view>
      <view class="flex  p-xs mb-sm">
        <view class="flex-sub bg-grey padding-sm margin-xs radius">设备详情地址:{{ workinfodata.deviceinfo}}</view>
      </view>
      <button @click="oktoorder(workinfodata.id)" style="width: 95%; margin-left: 20upx;" class="cu-btn block bg-blue margin-tb-sm lg">
        <text class="cuIcon-loading2 cuIconfont-spin" v-if="logsin"></text>我同意接单</button>
    </view>
    <!-- 等待去目的地位置 -->
    <view v-if="num == 1">
      <!-- 设备信息 -->
      <div class="hm-news-card">
        <div class="container">
          <div class="box">
            <map v-if="ismap" style="width: 100%; height: 550rpx;" v-for="(item,index) in covers" :key="index"
              :latitude="item.latitude" :longitude="item.longitude" :markers="covers">
            </map>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">设备编号号:{{ orderdata.devicenumber }}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">订单编号:{{ orderdata.order_num}}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">设备位置:{{ orderdata.location}}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">设备报警类型:{{ orderdata.status_name}}</view>
            <div class="submain" style="margin-top: 25rpx;margin-left: -15rpx;">
              <span class="date" style="font-size: 30rpx;color: #000000;">{{ orderdata.tname }}</span>
              <span class="date" style="font-size: 30rpx;color: #000000;">类型:{{ orderdata.gas }}</span>
              <span class="date" style="font-size: 30rpx;color: #000000;">浓度:{{ orderdata.concentration }}</span>
              <span class="date" style="font-size: 30rpx;color: #000000;">单位:{{ orderdata.danwei }}</span>
            </div>
            <div class="row_2" />
            <div class="ft">
              <button v-if="isdaobuton" style="margin-top: 10rpx;margin-left: 12upx;" class="cu-btn bg-green sm shadow"
                @click="gotolocation(orderdata.devicecoord)">导航去这里</button>
              <button v-if="oktobkorder" style="margin-top: 10rpx;margin-left: 12upx;" class="cu-btn bg-green sm shadow"
                @click="lsiokder(workinfodata.id)">下一步</button>
              <view v-if="nogettolis">
                <checkbox-group style="height: 100%;" @change="CheckboxChange">
                  <view style="margin-right: 280upx;margin-top: 8upx;">
                    <span>是否到达地点: </span>
                    <checkbox :class="isokfrom[0].checked?'checked':''" :checked="isokfrom[0].checked?true:false" value="1"></checkbox>
                  </view>
                  <view style="position:relative;top: -45upx; left: 260upx;" v-if="isokblak">
                    <span>现场是否属实:</span>
                    <checkbox :class="isokfrom[1].checked?'checked':''" :checked="isokfrom[1].checked?true:false" value="2"></checkbox>
                    <span v-if="isnobkil">不属实</span>
                    <checkbox v-if="isnobkil" :class="isokfrom[2].checked?'checked':''" :checked="isokfrom[2].checked?true:false"
                      value="3"></checkbox>
                  </view>
                </checkbox-group>
              </view>
              <!-- 别终止订单 -->
              <view v-if="istopsrty" style="margin-top: 10rpx;margin-left: 12upx;">
                <span>订单被终止-终止信息:</span>
                <span style="color:red">订单与现场不属实</span>
              </view>
            </div>
          </div>
        </div>
      </div>
      <!-- 不属实提交模态框 -->
      <view class="cu-modal" :class="modalName=='DialogModal1'?'show':''">
        <view class="cu-dialog">
          <view class="cu-bar bg-white justify-end">
            <view class="content">反馈信息</view>
            <view class="action" @tap="hideModal">
              <text class="cuIcon-close text-red"></text>
            </view>
          </view>
          <view>
            <view class="cu-form-group align-start">
              <textarea maxlength="-1" @input="textareaBInput" placeholder="反馈信息填写"></textarea>
            </view>
          </view>
          <view class="cu-bar bg-white justify-end">
            <view class="action">
              <button class="cu-btn line-green text-green" @tap="hideModal">取消</button>
              <button class="cu-btn bg-green margin-left" @tap="okhideModal(workinfodata.id)">确定</button>
            </view>
          </view>
        </view>
      </view>
    </view>
    <!-- 第三步 -->
    <view v-if="num == 2">
      <!-- 图片上传 -->
      <view v-if="istrwee">
        <view class="cu-bar bg-white">
          <view class="action">反馈图片</view>
          <view class="action">{{imgList.length}}/4</view>
        </view>
        <view class="cu-form-group">
          <view class="grid col-4 grid-square flex-sub">
            <view class="bg-img" v-for="(item,index) in imgList" :key="index" @tap="ViewImage" :data-url="imgList[index]">
              <image :src="imgList[index]" mode="aspectFill"></image>
              <view class="cu-tag bg-red" @tap.stop="DelImg" :data-index="index">
                <text class='cuIcon-close'></text>
              </view>
            </view>
            <view class="solids" @tap="ChooseImage" v-if="imgList.length<4">
              <text class='cuIcon-cameraadd'></text>
            </view>
          </view>
        </view>
        <!-- 修好或者没有修好 -->
        <view class="cu-form-group" style="margin-top: 0; margin-left: -20upx;">
          <view class="title" style="color: #8799A3; font-size: 30upx;">修好或未修好(默认修好)</view>
          <switch @change="SwitchA" :class="switchA?'checked':''" :checked="switchA?true:false"></switch>
        </view>
        <!-- 反馈信息 -->
        <view class="cu-form-group align-start" style="margin-left: -30rpx;">
          <view class="title" style="color: #8799A3; font-size: 30upx;">反馈信息</view>
          <textarea maxlength="-1" @input="textareaBInputfankui" placeholder="反馈信息填写"></textarea>
        </view>
        <!-- 提交 -->
        <view>
          <button style="width: 90%; margin-left: 30upx; margin-top: 60upx;" class="cu-btn block bg-blue margin-tb-sm lg"
            @click="toistrwee(workinfodata.id)">提交信息</button>
        </view>
      </view>
      <!-- 提交订单未审核时 -->
      <view v-if="okistrwee">
        <view class="wait_1">
          <view class="wait_12">
            <view class="wait_2" style="background-color: #F0F0F0;">
            </view>
            <image src="../../static/togif/shenhe.png" mode="scaleToFill" border="0" style="width: 100%;height: 100%;margin-left: 3rpx;margin-top: 5rpx;">
            </image>
            <text decode="true" class="wait_4">正努力审核,请稍后在查看....</text>
          </view>
        </view>
      </view>
    </view>
    <!-- 第四步 -->
    <view v-if="num == 3">
      <div class="hm-news-card">
        <div class="container">
          <div class="box">
            <map style="width: 100%; height: 550rpx;" v-for="(item,index) in covers" :key="index" :latitude="item.latitude"
              :longitude="item.longitude" :markers="covers">
            </map>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">设备编号号:{{ orderdata.devicenumber }}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">订单编号:{{ orderdata.order_num}}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">设备位置:{{ orderdata.location}}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">设备报警类型:{{ orderdata.status_name}}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">订单完成人:{{ orderdata.name}}</view>
            <view style="margin-top: 12rpx; margin-left: 20rpx;">订单完成时间:{{ orderdata.updated_at}}</view>
            <div class="submain" style="margin-top: 25rpx;margin-left: -15rpx;">
              <span class="date" style="font-size: 30rpx;color: #000000;">{{ orderdata.tname }}</span>
              <span class="date" style="font-size: 30rpx;color: #000000;">类型:{{ orderdata.gas }}</span>
              <span class="date" style="font-size: 30rpx;color: #000000;">浓度:{{ orderdata.concentration }}</span>
              <span class="date" style="font-size: 30rpx;color: #000000;">单位:{{ orderdata.danwei }}</span>
            </div>
          </div>
        </div>
      </div>
    </view>
  </view>
</template>

<script>
  import http from '@/components/utils/http.js';
  export default {
    data() {
      return {
        numList: [{
          name: '开始'
        }, {
          name: '等待'
        }, {
          name: '检修'
        }, {
          name: '完成'
        }],
        num: 0,
        modalName: null,
        workinfodata: '',
        logsin: false,
        istowork: true,
        istoworkrm: false,
        orderdata: '',
        covers: [{}],
        isokfrom: [{
          value: '1',
          checked: false
        }, {
          value: '2',
          checked: false
        }, {
          value: '3',
          checked: false
        }],
        isdaobuton: true,
        isokblak: false,
        isnobkil: true,
        oktobkorder: false,
        nogettolis: true,
        switchA: true, // 默认修好
        istopsrty: false,
        textareaBValue: '',
        textareaBInputfankuidata: '',
        imgList: [],
        istrwee: true, // 提交了工单 待审核
        isrepaired: 1, //默认修好
        okistrwee: false, //提交工单 未审核时
        ismap: true, //是否显示地图
        aram_orederid: '',
      }
    },
    methods: {
      NumSteps() {
        this.num = this.num == this.numList.length - 1 ? 0 : this.num + 1
      },
      // 获取工单详情 第一步
      workinfo(id) {
        uni.showLoading({
          title: '加载中'
        });
        let opts = {
          url: 'alarm_order/show',
          method: 'get'
        };
        let data = {
          id: id,
        };
        http.httpRequest(opts, data).then(res => {
          uni.hideLoading();
          this.workinfodata = res.data.data
        }, error => {
          console.log(error);
        })
      },
      oktoorder(id) {
        this.logsin = true;
        let data = {
          id: id,
          isorderone: 2,
          state: 1,
          schedule: JSON.stringify({
            'schedule': 2,
            'content': '提交工单' + id
          })
        };
        this.to_order_ok(data, 1);
      },
      //返回上一级页面
      toworklist() {
        uni.navigateTo({
          delta: 1,
        });
      },
      rmtoworklist() {
        uni.reLaunch({
          url: '../workorder/home'
        });
      },
      //监测订单状态
      istabto() {
        var isnavigateB = uni.getStorageSync('isnavigateB');
        if (isnavigateB == 1) {
          this.istowork = false;
          this.istoworkrm = true;
        }
      },
      CheckboxChange(e) {
        this.isfromin(e.detail.value);
        var items = this.isokfrom,
          values = e.detail.value;
        for (var i = 0, lenI = items.length; i < lenI; ++i) {
          items[i].checked = false;
          for (var j = 0, lenJ = values.length; j < lenJ; ++j) {
            if (items[i].value == values[j]) {
              items[i].checked = true;
              break
            }
          }
        }
      },
      // 图片
      ChooseImage() {
        uni.chooseImage({
          count: 4, //默认9
          sizeType: ['original', 'compressed'],
          sourceType: ['album'], //从相册选择
          success: (res) => {
            if (this.imgList.length != 0) {
              this.imgList = this.imgList.concat(res.tempFilePaths)
            } else {
              this.imgList = res.tempFilePaths
            }
            console.log(this.imgList);
          }
        });
      },
      ViewImage(e) {
        uni.previewImage({
          urls: this.imgList,
          current: e.currentTarget.dataset.url
        });
      },
      DelImg(e) {
        uni.showModal({
          title: '提示',
          content: '确定要删除这张图片吗？',
          cancelText: '再看看',
          confirmText: '再见',
          success: res => {
            if (res.confirm) {
              this.imgList.splice(e.currentTarget.dataset.index, 1)
            }
          }
        })
      },
      //提交工单
      toistrwee(id) {
        // 先提交图片
        if (this.imgList.length == 0) {
          uni.showToast({
            title: '请上传维修图片',
            icon: "none",
          });
          return false;
        }
        let header = {
          "Authorization": 'Bearer' + ' ' + uni.getStorageSync('Authorization'),
        };
        uni.showLoading({
          title: '加载中'
        });
        this.imgList.forEach((item, index) => {
          uni.uploadFile({
            url: http.baseUrl + 'upload_img/uploadingimg', //仅为示例，非真实的接口地址
            filePath: item,
            header: header,
            name: 'file',
            formData: {
              alarm_order_id: id,
            },
            success: (uploadFileRes) => {}
          });
        })
        this.fromdatatwree(id);
      },
      // 提交反馈信息 和修好未修好
      fromdatatwree(id) {
        let data = {
          id: id,
          isorderone: 3,
          state: 4,
          isrepaired: this.isrepaired,
          content: this.textareaBInputfankuidata,
          schedule: JSON.stringify({
            'schedule': 4,
            'content': '提交工单' + id
          })
        };
        this.to_order_ok(data, 4);
      },
      // 处理工单准备提交
      isfromin(data) {
        var arrlength = data.length;
        if (arrlength == 1) {
          if (data.indexOf(1)) {
            this.isdaobuton = false; //这里代表她已经到达地点
            this.isokblak = true;
            this.oktobkorder = false;
            this.isnobkil = true;
          }
          if (data.indexOf(3)) { //代表不属实
            this.isdaobuton = false; //这里代表她已经到达地点
            this.isokblak = true;
          }
          if (data[0] == 2) { //这里代表 点击属实去掉了到达地点
            this.isnobkil = false;
            this.isdaobuton = true;
            this.oktobkorder = false;
            this.isokblak = false;
          }
        }
        if (arrlength == 0) {
          this.isdaobuton = true;
          this.isokblak = false;
        }
        if (arrlength == 2) { //代表这里属实 要进行下一步
          if (data[0] == 1 && data[1] == 2) { //代表这里属实 要进行下一步
            this.isdaobuton = false;
            this.isokblak = true;
            this.isnobkil = false;
            this.oktobkorder = true;
          }
        }
        if (arrlength == 2) { //代表这里不属实 要进行下一步 提交 
          if (data[0] == 1 && data[1] == 3) { //代表这里属实 要进行下一步
            //打开模态框
            this.ismap = false;
            this.showModal();
          }
        }
      },
      showModal(e) {
        this.modalName = 'DialogModal1';
      },
      hideModal(e) {
        this.modalName = null;
        // 去掉不属实选择
        this.isokfrom[2].checked = false;
        this.ismap = true;
      },
      textareaBInput(e) {
        this.textareaBValue = e.detail.value
      },
      textareaBInputfankui(e) {
        this.textareaBInputfankuidata = e.detail.value
      },
      SwitchA(e) {
        this.switchA = e.detail.value
        this.isrepaired = e.detail.value ? '1' : '2';
      },
      // 处理不属实订单提交反馈信息
      okhideModal(id) {
        let data = {
          id: id,
          isorderone: 2,
          state: 3,
          is_verified: 2,
          is_live: 1,
          content: this.textareaBValue,
          schedule: JSON.stringify({
            'schedule': 2,
            'content': '工单与说明不属实' + id
          })
        };
        this.to_order_ok(data, 2)
      },
      //提交进行下一步 这里指的是第三步
      lsiokder(id) {
        let data = {
          id: id,
          isorderone: 3,
          state: 1,
          is_verified: 1,
          is_live: 1,
          schedule: JSON.stringify({
            'schedule': 3,
            'content': '工单到达检修' + id
          })
        };
        this.to_order_ok(data, 3)
      },
      // 导航开启 采用手机导航
      gotolocation(Latitudeandlongitude) {
        var newlatitude = Latitudeandlongitude.split(",")
        uni.openLocation({
          latitude: Number(newlatitude[1]),
          longitude: Number(newlatitude[0]),
          success: function() {}
        });
      },
      // 监测当前订单为第几步
      isorderone(id) {
        let opts = {
          url: 'alarm_order/order_in',
          method: 'get'
        };
        let data = {
          id: id,
        };
        http.httpRequest(opts, data).then(res => {
          switch (res.data.data.isorderone) {
            case 1:
              this.workinfo(res.data.data.id);
              this.num = 0;
              break;
            case 2:
              this.tolocation(res.data.data.id);
              this.num = 1;
              break;
            case 3:
              this.tolocation(res.data.data.id);
              this.num = 2;
              break;
            case 4:
              this.tolocation(res.data.data.id);
              this.num = 3;
              break;
            default:
          }
        }, error => {
          console.log(error);
        })
      },
      //第二步
      tolocation(id) {
        let opts = {
          url: 'alarm_order/show',
          method: 'get'
        };
        let data = {
          id: id,
        };
        this.isdaobuton = false;
        this.nogettolis = false;
        this.istrwee = false;
        http.httpRequest(opts, data).then(res => {
          if (res.data.data.state == 4) {
            this.imgList = [];
            this.isrepaired = 1;
            this.istrwee = false;
            this.okistrwee = true;
            return false;
          }
          if (res.data.data.state == 3) { //工单异常终止
            this.isdaobuton = false;
            this.nogettolis = false;
            this.istopsrty = true;
            this.workinfodata = res.data.data
            this.orderdata = res.data.data;
            var newlatitude = res.data.data.devicecoord.split(",")
            this.covers.forEach((item, index) => {
              this.$set(item, 'latitude', newlatitude[1])
              this.$set(item, 'longitude', newlatitude[0])
              this.$set(item, 'iconPath', '../../static/location.png')
            })
          } else {
            this.isdaobuton = true;
            this.nogettolis = true;
            this.istrwee = true;
            this.workinfodata = res.data.data
            this.orderdata = res.data.data;
            var newlatitude = res.data.data.devicecoord.split(",")
            this.covers.forEach((item, index) => {
              this.$set(item, 'latitude', newlatitude[1])
              this.$set(item, 'longitude', newlatitude[0])
              this.$set(item, 'iconPath', '../../static/location.png')
            })
          }
        }, error => {
          console.log(error);
        })
      },
      //修改订单公共方法
      to_order_ok(data, type) {
        // type == 1 我去接单这里提交
        // type == 2 我是不属实订单提交
        uni.showLoading({
          title: '加载中'
        });
        let opts = {
          url: 'alarm_order/order_ok',
          method: 'post'
        };
        http.httpRequest(opts, data).then(res => {
          if (type == 1) {
            this.isdaobuton = false;
            this.nogettolis = false;
            if (res.data.code == 200) {
              uni.hideLoading();
              this.logsin = false;
              this.NumSteps();
              this.isorderone(data.id);
              uni.setStorageSync('isnavigateB', 1); //缓存则 1 == 接单以后点击返回按键 或者 点击返回按钮 返回 工单列表页 
            } else {
              this.logsin = false;
              uni.showToast({
                title: '失败请重试',
                duration: 2000
              });
            }
          } else if (type == 2) {
            if (res.data.code == 200) {
              uni.hideLoading();
              this.isdaobuton = false;
              this.nogettolis = false;
              this.istopsrty = true;
              this.hideModal();
            }
          } else if (type == 3) { //这里是检修 第三步
            if (res.data.code == 200) {
              uni.hideLoading();
              this.isdaobuton = false;
              this.nogettolis = false;
              this.NumSteps();
              this.tolocation(data.id);
            }
          } else if (type == 4) {
            uni.hideLoading();
            if (res.data.code == 200) {
              this.imgList = [];
              this.isrepaired = 1;
              this.istrwee = false;
              this.okistrwee = true;
            }
          }
        }, error => {
          console.log(error);
        })
      },
    },
    onLoad(id) {
      this.isorderone(id.id); //获取 当前订单为第几部
      this.aram_orederid = id;
    },
    created() {
      this.istabto();
    },
    //监听返回按键
    onBackPress() {
      var isnavigateB = uni.getStorageSync('isnavigateB');
      if (isnavigateB == 1) { //这里监听返回按键 1 == 代表已经接单 工单正在进行中 点击返回键 返回工单列表
        uni.navigateBack({
          delta: 1,
        });
      } else {
        this.toworklist();
      }
    },
    //下拉刷新
    onPullDownRefresh() {
      this.istabto();
      this.isorderone(this.aram_orederid.id); //获取 当前订单为第几部
      uni.stopPullDownRefresh();
    },
  }
</script>

<style lang="scss" scoped>
  @import './index.response.css';
  @import './wait.scss';
</style>
