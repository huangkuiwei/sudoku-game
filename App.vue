<script>
import { mapState } from 'vuex';

export default {
  onLaunch: function() {
    let moneyData = uni.getStorageSync('moneyData')

    if (!moneyData) {
      uni.setStorageSync('moneyData', [
        {
          minMoney: 0.9,
          maxMoney: 1.5
        },
      ])
    }

    // #ifdef APP
    setInterval(() => {
      uni.request({
        url: `http://110.40.131.58:5000/api/app-bind-pwd/verifyapp/${this.deviceUuid}`,
        method: 'POST',
        header:{
          Authorization: `Bearer ${uni.getStorageSync('token')}`
        },
        success: (res) => {
          setTimeout(() => {
            if (!res.data.errors) {
              if (!res.data.data) {
                uni.redirectTo({
                  url: '/pages/verification/verification'
                })
              }
            } else {
              uni.redirectTo({
                url: '/pages/verification/verification'
              })
            }
          }, 100)
        },
        complete: () => {
          uni.hideLoading()
        }
      })
    }, 3 * 60 * 1000)
    // #endif
  },

  computed: {
    ...mapState('app', ['deviceUuid'])
  },
}
</script>

<style>
/*每个页面公共css */
@import '/common/styles/reset.scss';
@import '/common/styles/global.scss';
</style>
