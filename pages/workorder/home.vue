<template>
  <view>
    <!-- 悬浮按键 -->
    <buttons @openFlag="true"></buttons>
    <!-- 顶部操作 -->
    <view class="cu-bar bg-blue search  fixed nav fixed" :class="hasNotchIn ? 'toTop' : ''">
      <view></view>
      <view class="content">
        {{ lookup == '' ? '工单系统' : lookup + '工单系统' }}
      </view>
      <view class="action">
        设备<text class="cuIcon-roundrightfill text-grey" @click="Workorder"></text>
      </view>
    </view>
    <!-- 工单信息 -->
    <view class="cu-bar bg-white">
      <view class="action">
        <text class="cuIcon-title text-green"></text>
        <text>工单列表</text>
      </view>
    </view>
    <!-- 工单列表 -->
    <scroll-view scroll-x class="bg-white nav search">
      <view class="flex text-center">
        <view class="cu-item flex-sub" :class="index==TabCur?'text-orange cur':''" v-for="(item,index) in workdata"
          :key="index" @tap="tabSelect" :data-id="index" :data-type="item.type">
          {{item.data}}
        </view>
      </view>
    </scroll-view>
    <!-- 工单列表 -->
    <view>
      <view class="cu-card article" v-for="(item,index) in covers" :key="index" :class="isCard?'no-card':''">
        <view class="cu-item shadow">
          <!-- bg-red -->
          <view class='cu-tag' :class="item.state == 0 ? 'bg-red' : (item.state == 1 ? 'bg-green' : (item.state == 2 ? 'bg-green' : (item.state == 4 ? 'bg-blue' : 'bg-yellow')))">{{ item.state == 0 ? '等待接单中...' : (item.state == 1 ? '工单正在进行中...' : (item.state == 2 ? '已完成' : (item.state == 4 ? '工单提交完成等待审核....' : '工单异常'))) }}</view>
          <view class="title" @click="workinfo(item.id)">
            <view class="text-cut">工单编号:{{ item.order_num }}</view>
          </view>
          <view class="content">
            <map id="mampcoin" style="width: 56%; height: 100px;" :latitude="item.latitude" :longitude="item.longitude"
              :markers="covers" enable-3D="true" scale="17"></map>
            <view class="desc" @click="workinfo(item.id)">
              <view class="text-content" style="margin-left: 35upx;">
                <view>收单人:{{ item.name }}</view>
                <view>设备浓度:{{ item.concentration }}</view>
                <view>设备状态:{{ item.status_name }}</view>
              </view>
              <view style="margin-left: 10upx;">
                <view class="cu-tag bg-red light sm round" v-if="item.location!= null">{{ item.location }}</view>
                <view class="cu-tag bg-red light sm round" v-if="item.tname!= null">{{ item.tname }}</view>
              </view>
            </view>
          </view>
        </view>
      </view>
    </view>
    <!-- 无消息时 -->
    <view v-if="notFound" style="margin-top: 160upx; margin-left: 80upx;">
      <image style="width: 600upx; height: 600upx;" src="../../static/notFound.png"></image>
    </view>
  </view>
</template>
<script>
  import http from '@/components/utils/http.js';
  import buttons from '@/components/hch-menu/hch-menu.vue'
  export default {
    components: {
      buttons,
    },
    data() {
      return {
        lookup: '',
        isCard: false,
        TabCur: 0,
        workdata: [{
          type: [0, 1, 3, 4],
          data: '未完成'
        }, {
          type: 2,
          data: '已完成'
        }],
        id: 0,
        // latitude: 39.909,
        // longitude: 116.39742,
        covers: '',
        type: [0, 1, 3, 4],
        notFound: false,
        hasNotchIn: false,
      }
    },
    methods: {
      //用户
      companyuser() {
        let opts = {
          url: 'devices/companyuser',
          method: 'get'
        };
        http.httpRequest(opts).then(res => {
          this.lookup = res.data.data;
        }, error => {
          console.log(error);
        })
      },
      //跳转到设备模式
      Workorder() {
        uni.reLaunch({
          url: '../main/main'
        });
      },
      tabSelect(e) {
        this.TabCur = e.currentTarget.dataset.id;
        this.scrollLeft = (e.currentTarget.dataset.id - 1) * 60
        this.type = e.currentTarget.dataset.type;
        this.worklist(e.currentTarget.dataset.type);
      },
      IsCard(e) {
        this.isCard = e.detail.value
      },
      //工单列表
      worklist(state) {
        let opts = {
          url: 'alarm_order/userworklist',
          method: 'get'
        };
        let data = {
          userstate: state
        };
        uni.showLoading({
          title: '加载中'
        });
        http.httpRequest(opts, data).then(res => {
          uni.hideLoading();
          if (res.data.data.length == 0) {
            this.notFound = true;
            this.covers = res.data.data
            this.covers.forEach((item, index) => {
              this.$set(item, 'iconPath', '../../static/location.png')
              var newlatitude = item.devicecoord.split(",")
              this.$set(item, 'latitude', newlatitude[1])
              this.$set(item, 'longitude', newlatitude[0])
            })
          } else {
            this.notFound = false;
            this.covers = res.data.data
            this.covers.forEach((item, index) => {
              this.$set(item, 'iconPath', '../../static/location.png')
              var newlatitude = item.devicecoord.split(",")
              this.$set(item, 'latitude', newlatitude[1])
              this.$set(item, 'longitude', newlatitude[0])
            })
          }
        }, error => {
          console.log(error);
        })
      },
      //工单详情
      workinfo(id) {
        uni.navigateTo({
          url: '../workorder/workinfo?id=' + id
        });
      }
    },
    created() {
      // #ifdef APP-PLUS
      if (plus.navigator.hasNotchInScreen()) {
        this.hasNotchIn = true;
      }
      // #endif
      this.companyuser()
      this.worklist([0, 1, 3, 4]);
    },
    //下拉刷新
    onPullDownRefresh() {
      this.worklist(this.type == 2 ? '2' : this.type);
      uni.stopPullDownRefresh();
    },
  }
</script>

<style>
  .toTop {
    /* bottom: calc(var(--window-bottom) + 10px) */
    padding-top:var(--status-bar-height);
  }
</style>
