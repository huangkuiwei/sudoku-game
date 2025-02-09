<template>
  <view class="container">
    <view class="title">
      <view class="grade">
        第{{ currentBoardIndex + 1 }}关
      </view>

      <view class="options">
        <view class="left" @click="$refs.withdrawalDialogRef.open()">
          <image mode="widthFix" src="/static/images/weixin_pay_icon.png"/>
          <text>提现</text>
        </view>

        <view class="time">时间{{ convertSecondsToMMSS(countdown) }}</view>

        <view class="right">
          <image mode="widthFix" src="/static/images/hongbao.png"/>
          <text>{{ totalMoney }}</text>
        </view>
      </view>
    </view>

    <view>
      <view class="sudoku-board">
        <view v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
          <view
              v-for="(cell, colIndex) in row"
              :key="colIndex"
              class="cell"
              :class="{ 'filled': cell !== 0, 'selected': selectedRow === rowIndex && selectedCol === colIndex && !isFixed(rowIndex, colIndex) }"
              @click="selectCell(rowIndex, colIndex)"
          >
            {{ cell === 0 ? '' : cell }}
          </view>
        </view>
      </view>

      <view class="tip">填入数字，保证每一行列的数字不重复</view>

      <view class="controls">
        <button
            v-for="num in numbers"
            :key="num"
            class="number-button"
            @click="enterNumber(num)"
        >
          {{ num }}
        </button>
      </view>
    </view>

    <uni-popup ref="hongbaoRef" :mask-click="false" background-color="#ffffff" border-radius="5px 5px 5px 5px">
      <view class="hongbao-dialog">
        <view class="title">恭喜获得红包</view>
        <view class="money1">
          <image mode="widthFix" src="/static/images/hongbao.png"/>
          <text>￥{{ money }}</text>
        </view>
        <view class="money2">已自动存入余额</view>

        <view class="btn">
          <button type="primary" size="mini" @click="$refs.hongbaoRef.close(); isComplete && generateNextLevel()">
            {{ isComplete ? '下一关' : '太棒了' }}
          </button>
        </view>
      </view>
    </uni-popup>

    <uni-popup ref="failRef" :mask-click="false" background-color="#ffffff" border-radius="5px 5px 5px 5px">
      <view class="hongbao-dialog">
        <view class="title" style="margin-bottom: 40rpx">时间到，游戏结束</view>
        <view class="btn">
          <button type="primary" size="mini" @click="$refs.failRef.close(); generateNextLevel()">下一关</button>
        </view>
      </view>
    </uni-popup>

    <uni-popup ref="withdrawalDialogRef" background-color="#ffffff" border-radius="5px 5px 5px 5px">
      <view class="withdrawal-dialog">
        <view class="title">提现</view>
        <input type="number" v-model="withdrawalMoney" placeholder="请输入提现金额">
        <view class="btn">
          <button @click="$refs.withdrawalDialogRef.close()">
            取消
          </button>

          <button type="primary" @click="withdrawal">
            确定
          </button>
        </view>
      </view>
    </uni-popup>

    <view class="notice-dialog" v-if="showNoticeDialog" @click="jumpUrl('/pages/noticeList/noticeList')">
      <view class="left">
        <image mode="widthFix" src="/static/images/horn.png"/>
      </view>

      <view class="right">
        <text>微信支付</text>
        <text>微信支付：您收到一笔活动奖励</text>
      </view>
    </view>
  </view>
</template>

<script>
import { mapState } from 'vuex'

export default {
  data() {
    return {
      completeBoards: [
        // Pre-generated valid Sudoku boards
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [4, 3, 5, 2, 6, 9, 7, 8, 1],
          [6, 8, 2, 5, 7, 1, 4, 9, 3],
          [1, 9, 7, 8, 3, 4, 5, 6, 2],
          [8, 2, 6, 1, 9, 5, 3, 4, 7],
          [3, 7, 4, 6, 8, 2, 9, 1, 5],
          [9, 5, 1, 7, 4, 3, 6, 2, 8],
          [5, 1, 9, 3, 2, 6, 8, 7, 4],
          [2, 4, 8, 9, 5, 7, 1, 3, 6],
          [7, 6, 3, 4, 1, 8, 2, 5, 9]
        ],
        [
          [8, 2, 7, 1, 5, 4, 3, 9, 6],
          [9, 6, 5, 3, 2, 7, 4, 8, 1],
          [3, 4, 1, 6, 8, 9, 5, 2, 7],
          [5, 9, 6, 4, 7, 3, 8, 1, 2],
          [4, 7, 2, 8, 1, 5, 6, 3, 9],
          [1, 3, 8, 9, 6, 2, 7, 4, 5],
          [6, 1, 4, 7, 9, 8, 2, 5, 3],
          [2, 5, 3, 4, 8, 1, 9, 6, 7],
          [7, 8, 9, 2, 3, 6, 1, 4, 8]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ],
        [
          [5, 3, 4, 6, 7, 8, 9, 1, 2],
          [6, 7, 2, 1, 9, 5, 3, 4, 8],
          [1, 9, 8, 3, 4, 2, 5, 6, 7],
          [8, 5, 9, 7, 6, 1, 4, 2, 3],
          [4, 2, 6, 8, 5, 3, 7, 9, 1],
          [7, 1, 3, 9, 2, 4, 8, 5, 6],
          [9, 6, 1, 5, 3, 7, 2, 8, 4],
          [2, 8, 7, 4, 1, 9, 6, 3, 5],
          [3, 4, 5, 2, 8, 6, 1, 7, 9]
        ]
      ],
      currentBoardIndex: 0,
      initialBoard: [],
      board: [],
      selectedRow: -1,
      selectedCol: -1,
      numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9],
      isComplete: false,
      countdown: 0,
      money: 0,
      totalMoney: Number(localStorage.getItem('totalMoney')) || 0,
      withdrawalMoney: 0,
      showNoticeDialog: false
    }
  },
  created() {
    this.generatePuzzle()
  },

  computed: {
    ...mapState('app', ['noticeList'])
  },

  watch: {
    countdown: {
      handler(value) {
        if (value) {
          setTimeout(() => {
            this.countdown -= 1
          }, 1000)
        } else {
          // 游戏结束
          this.$refs.failRef.open()
        }
      },
    }
  },

  methods: {
    convertSecondsToMMSS(seconds) {
      const minutes = Math.floor(seconds / 60)
      const remainingSeconds = seconds % 60
      const formattedMinutes = String(minutes).padStart(2, '0')
      const formattedSeconds = String(remainingSeconds).padStart(2, '0')

      return `${formattedMinutes}:${formattedSeconds}`
    },

    generatePuzzle() {
      if (this.currentBoardIndex >= this.completeBoards.length) {
        uni.showToast({
          title: '通关完成',
          icon: 'success'
        })
        return
      }

      this.initialBoard = JSON.parse(JSON.stringify(this.completeBoards[this.currentBoardIndex]))
      this.board = JSON.parse(JSON.stringify(this.initialBoard))
      this.removeNumbers(this.board, 40) // Remove 40 numbers to create a puzzle
      this.selectedRow = -1
      this.selectedCol = -1
      this.isComplete = false
      this.countdown = 10 * 60
    },

    selectCell(row, col) {
      if (this.isFixed(row, col)) return
      this.selectedRow = row
      this.selectedCol = col
    },

    enterNumber(num) {
      if (this.selectedRow >= 0 && this.selectedCol >= 0) {
        if (this.isValid(this.board, this.selectedRow, this.selectedCol, num)) {
          this.$set(this.board[this.selectedRow], this.selectedCol, num)
          this.selectedRow = -1
          this.selectedCol = -1
          if (this.checkCompletion()) {
            this.isComplete = true
            setTimeout(() => {
              this.currentBoardIndex++
              this.generatePuzzle()
            }, 2000)
          }

          let moneyData = uni.getStorageSync('moneyData')[0]
          let money = Number(moneyData.minMoney) + Math.random() * (moneyData.maxMoney - moneyData.minMoney)
          this.money = Number(money.toFixed(2))
          this.totalMoney = Number((this.totalMoney + this.money).toFixed(2))
          this.$refs.hongbaoRef.open()

          localStorage.setItem('totalMoney', this.totalMoney)
        } else {
          uni.showToast({
            title: '填入错误',
            icon: 'error'
          })
        }
      }
    },

    isValid(board, row, col, num) {
      return this.initialBoard[row][col] === num
    },

    removeNumbers(board, count) {
      let removedCount = 0
      while (removedCount < count) {
        const row = Math.floor(Math.random() * 9)
        const col = Math.floor(Math.random() * 9)
        if (board[row][col] !== 0) {
          board[row][col] = 0
          removedCount++
        }
      }
    },

    isFixed(row, col) {
      return this.board[row][col] !== 0
    },

    checkCompletion() {
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (this.board[row][col] === 0 || this.board[row][col] !== this.initialBoard[row][col]) {
            return false
          }
        }
      }
      return true
    },

    generateNextLevel() {
      this.currentBoardIndex++
      this.generatePuzzle()
    },

    withdrawal() {
      if (Number(this.withdrawalMoney) < 1) {
        uni.showToast({
          title: '最低提现1',
          icon: 'error'
        })

        return
      }

      if (Number(this.withdrawalMoney) > this.totalMoney) {
        uni.showToast({
          title: '余额不足',
          icon: 'error'
        })

        return
      }

      uni.showLoading({
        title: '请稍等...',
        mask: true,
      })

      setTimeout(() => {
        uni.showToast({
          title: '提现成功',
          icon: 'success'
        })

        this.$refs.withdrawalDialogRef.close()
        this.totalMoney = Number((this.totalMoney - this.withdrawalMoney).toFixed(2))
        localStorage.setItem('totalMoney', this.totalMoney)

        setTimeout(() => {
          this.showNoticeDialog = true

          this.noticeList.push({
            time: Date.now(),
            money: Number(this.withdrawalMoney).toFixed(2),
          })

          setTimeout(() => {
            this.showNoticeDialog = false
          }, 4000)
        }, 4000)
      }, 2000)
    },

    jumpUrl(url) {
      uni.navigateTo({
        url: url
      })
    }
  },
}
</script>

<style lang="scss">
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
  height: 100vh;
  background: url("/static/images/bg.png") no-repeat top left/100% 100%;

  .title {
    align-self: stretch;

    .grade {
      text-align: center;
      margin-bottom: 20rpx;
    }

    .options {
      padding: 0 40rpx;
      display: flex;
      align-items: center;
      justify-content: space-between;

      .left, .right {
        font-size: 22rpx;
        display: flex;
        align-items: center;
        gap: 10rpx;
        background: #ffffff;
        padding: 4rpx 16rpx;
        border-radius: 16rpx;

        &.left {
          color: #00c800;
        }

        &.right {
          color: red;
        }

        image {
          width: 40rpx;
        }
      }

      .time {
        font-size: 28rpx;
      }
    }
  }
}

.sudoku-board {
  display: flex;
  flex-direction: column;
  border-left: 2rpx solid #000000;
  border-top: 2rpx solid #000000;
}

.row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;

  &:nth-child(4) {
    border-top: 2rpx solid #000000;
  }

  &:nth-child(7) {
    border-top: 2rpx solid #000000;
  }
}

.cell {
  border-right: 2rpx solid #000000;
  border-bottom: 2rpx solid #000000;
  width: 74rpx;
  height: 74rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40rpx; /* Increased font size */
  cursor: pointer;
  position: relative;

  &:nth-child(4) {
    border-left: 2rpx solid #000000;
  }

  &:nth-child(7) {
    border-left: 2rpx solid #000000;
  }
}

.cell.filled {
  background-color: #f0f0f0;
}

.cell.selected {
  background-color: #add8e6;
}

.cell.invalid {
  background-color: #ffcccc;
}

.controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
}

.tip {
  padding: 40rpx 0 20rpx;
  text-align: center;
}

.number-button {
  width: 74rpx;
  height: 74rpx;
  border: 2rpx solid #000000;
  font-size: 48rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.error-message {
  color: red;
  margin-top: 20rpx;
  width: 90%;
  max-width: 630rpx; /* Adjusted for better visual appearance */
  text-align: center;
}

.success-message {
  color: green;
  margin-top: 20rpx;
  width: 90%;
  max-width: 630rpx; /* Adjusted for better visual appearance */
  text-align: center;
}

.hongbao-dialog {
  width: 550rpx;
  padding: 40rpx 50rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  .title {
    text-align: center;
    font-size: 36rpx;
  }

  .money1 {
    position: relative;

    image {
      width: 300rpx;
    }

    text {
      position: absolute;
      font-size: 40rpx;
      color: #e0f868;
      bottom: 60rpx;
      left: 0;
      right: 0;
      text-align: center;
    }
  }

  .money2 {
    font-size: 28rpx;
    margin-bottom: 10rpx;
  }

  .btn {
    align-self: stretch;
    display: flex;
    align-items: center;

    button {
      width: 70%;
    }
  }
}

.withdrawal-dialog {
  width: 550rpx;
  padding: 40rpx 50rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  .title {
    text-align: center;
    font-size: 40rpx;
    font-weight: bold;
    margin-bottom: 40rpx;
  }

  input {
    height: 86rpx;
    margin-bottom: 40rpx;
    width: 100%;
    border: 1px solid #cccccc;
    border-radius: 16rpx;
    padding: 0 30rpx;
    font-size: 36rpx;
  }

  .btn {
    align-self: stretch;
    display: flex;
    align-items: center;
    justify-content: space-around;

    button {
      width: 180rpx;
      height: 70rpx;
      line-height: 70rpx;
      font-size: 30rpx;
    }
  }
}

.notice-dialog {
  display: flex;
  align-items: center;
  background: #ffffff;
  position: fixed;
  z-index: 999;
  left: 40rpx;
  right: 40rpx;
  border-radius: 8rpx;
  padding: 30rpx 30rpx;
  top: 0;
  opacity: 0;
  transition: all 0.3s ease;
  animation: 4s ease kf1;

  @keyframes kf1 {
    0% {
      top: 0;
      opacity: 0;
    }

    10% {
      top: 100rpx;
      opacity: 1;
    }

    90% {
      top: 100rpx;
      opacity: 1;
    }

    100% {
      top: 0;
      opacity: 0;
    }
  }

  .left {
    background: #00C800;
    width: 90rpx;
    height: 90rpx;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 40rpx;

    image {
      width: 50rpx;
    }
  }

  .right {
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 10rpx;

    text {
      &:nth-child(1) {
        font-size: 32rpx;
        font-weight: bold;
      }

      &:nth-child(2) {
        font-size: 28rpx;
      }
    }
  }
}
</style>
