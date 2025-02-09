<script>
import { mapState } from 'vuex'

export default {
  name: 'noticeList',

  computed: {
    ...mapState('app', ['noticeList'])
  },

  methods: {
    convertTimestampToHHMM(timestamp) {
      const date = new Date(timestamp);
      const hours = String(date.getHours()).padStart(2, '0');
      const minutes = String(date.getMinutes()).padStart(2, '0');

      return `${hours}:${minutes}`;
    },

    back() {
      uni.navigateBack()
    }
  }
}
</script>

<template>
  <view class="notice-list">
    <view class="navbar">
      <image class="back" mode="widthFix" src="/static/images/arrow.png" @click="back"/>
      <view class="title">微信支付</view>
      <view class="right">
        <image class="fdj" mode="widthFix" src="/static/images/fdj.png"/>
        <image class="fdj" mode="widthFix" src="/static/images/setting.png"/>
      </view>
    </view>

    <view class="list">
      <view class="item" v-for="(item, index) of noticeList" :key="index">
        <view class="time">{{ convertTimestampToHHMM(item.time) }}</view>
        <view class="detail">
          <view class="top">
            <image mode="widthFix" src="/static/images/weixin_pay_icon.png"/>
            <text>微信支付</text>
          </view>

          <view class="bottom">
            <view>收款通知-来自活动奖励</view>
            <view>收款金额</view>
            <view>￥{{ item.money }}</view>
            <view>
              <text>收款账户</text>
              <text>零钱</text>
            </view>
            <view>
              <text>红包说明</text>
              <text>收益发放</text>
            </view>
            <view>
              <text>查看账单详情</text>
              <text>></text>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<style lang="scss">
page {
  background: #EDEDED;
}

.notice-list {
  padding-top: 40rpx;

  .navbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20rpx 40rpx;
    position: relative;

    image {
      width: 40rpx;

      &.back {
        transform: rotate(180deg);
      }
    }

    .title {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      text-align: center;
    }

    .right {
      display: flex;
      align-items: center;
      gap: 28rpx;
    }
  }

  .list {
    display: flex;
    flex-direction: column;
    gap: 20rpx;

    .item {
      align-self: stretch;
      width: 100%;
      padding: 0 40rpx;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: #1A1A1A;

      .time {
        background: #F7F7F7;
        border-radius: 6rpx;
        padding: 4rpx 10rpx;
        font-size: 24rpx;
        margin-bottom: 20rpx;
      }

      .detail {
        width: 100%;
        background: #ffffff;
        border-radius: 12rpx;

        .top {
          padding: 30rpx;
          border-bottom: 1px solid #EDEDED80;
          display: flex;
          align-items: center;
          gap: 10rpx;

          image {
            width: 40rpx;
          }
        }

        .bottom {
          padding: 40rpx 40rpx 0;
          font-size: 28rpx;

          view {
            &:nth-child(1) {
              margin-bottom: 20rpx;
            }

            &:nth-child(2) {
              text-align: center;
              font-size: 24rpx;
              color: #aaa;
              margin-bottom: 30rpx;
            }

            &:nth-child(3) {
              text-align: center;
              font-size: 50rpx;
              margin-bottom: 30rpx;
            }

            &:nth-child(4) {
              margin-bottom: 30rpx;
              display: flex;
              align-items: center;
              gap: 30rpx;

              text {
                &:nth-child(1) {
                  color: #aaa;
                }
              }
            }

            &:nth-child(5) {
              margin-bottom: 30rpx;
              display: flex;
              align-items: center;
              gap: 30rpx;

              text {
                &:nth-child(1) {
                  color: #aaa;
                }
              }
            }

            &:nth-child(6) {
              padding-bottom: 20rpx;
              display: flex;
              align-items: center;
              justify-content: space-between;
            }
          }
        }
      }
    }
  }
}
</style>
