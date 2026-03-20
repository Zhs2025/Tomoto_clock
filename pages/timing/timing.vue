<template>
  <view class="countdown-page">
    <!-- ==================== 顶部一级子容器（占一半高度） ==================== -->
    <view class="top-section">
      <!-- 居中圆形 -->
      <view class="circle-box">
        <!-- 上部二级子容器：显示标题 givenText -->
        <view class="circle-title">{{ givenText }}</view>

        <!-- 中部二级子容器：倒计时时间（实时更新） -->
        <view class="circle-time">{{ showTime }}</view>

        <!-- 底部二级子容器：预计结束时间（时分） -->
        <view class="circle-end">结束于 {{ endTime }}</view>
      </view>
    </view>

    <!-- ==================== 底部一级子容器 ==================== -->
    <view class="bottom-section">
      <!-- 左侧按钮：关闭 × -->
      <view class="btn-round" @click="goBack">
        <text class="close-icon">×</text>
      </view>

      <!-- 右侧按钮：播放 / 暂停 切换 -->
      <view class="btn-round" @click="togglePlay">
        <text v-if="!isPaused" class="play-icon">||</text>
        <text v-else class="pause-icon">▸</text>
      </view>
    </view>
  </view>
</template>




<script>
	
// 新增：【页面最顶部】音频实例（唯一）
// 由于是全局变量，必须在onunload 和 onHide 中 释放
const innerAudioContext = uni.createInnerAudioContext();

export default {
  data() {
    return {
      // 接收的参数
      givenText: "",
      selectedTime: "", // 格式 0:0:5 或 00:01:05

      // 倒计时核心
      totalSeconds: 0,
      remainSeconds: 0,
      showTime: "00:00:00",
      endTime: "",

      // 播放状态
      isPaused: false,
      timer: null,
	  // 默认背景音乐路径
	  bgAudioSrc: "/static/bubbling-brook2.mp3" 
    };
  },

  onLoad(options) {
    // 接收参数
        this.givenText = decodeURIComponent(options.inputText || '')
        this.selectedTime = decodeURIComponent(options.selectedTime || '00:00:10')
    
        // 🔥 关键：强制拆分 + 转数字，彻底杜绝 NaN
        let [h = 0, m = 0, s = 0] = this.selectedTime.split(':')
        h = Number(h) || 0
        m = Number(m) || 0
        s = Number(s) || 0
        this.selectedTime = `${h}:${m}:${s}`
    
        // 初始化倒计时
        this.initCountdown()
        this.startCountdown()
  },

	onUnload() {
		// 页面销毁清除定时器
		if (this.timer) {
		  clearInterval(this.timer);
		  this.timer = null;
		}
		// 页面销毁清除背景音乐
		this.stopBGM();
	},
	
	onHide() {
		// 页面隐藏清除背景音乐
		this.stopBGM();
	},
	
  methods: {
    // 初始化时间
    initCountdown() {
      // 拆分时分秒
      let [h, m, s] = this.selectedTime.split(":");
      h = Number(h);
      m = Number(m);
      s = Number(s);

      // 总秒数
      this.totalSeconds = h * 3600 + m * 60 + s;
      this.remainSeconds = this.totalSeconds;

      // 显示格式化时间
      this.showTime = this.formatTime(h, m, s);

      // 计算结束时间（当前 + 倒计时，只显示时分）
      this.calcEndTime();
    },

    // 开始倒计时
    startCountdown() {
		if (this.timer) clearInterval(this.timer);

		// 开始倒计时调用播放背景音乐函数
		this.playBGM();
		this.timer = setInterval(() => {
			if (this.isPaused || this.remainSeconds <= 0) {
				if (this.remainSeconds <= 0) {
					clearInterval(this.timer);
					this.showTime = "00:00:00";
					// 倒计时结束  停止音乐
					this.stopBGM();
				}
				return;
			}

			this.remainSeconds--;
			this.showTime = this.secToTime(this.remainSeconds);
		}, 1000);
		
	},

    // 暂停 / 播放切换
    togglePlay() {
		this.isPaused = !this.isPaused;
		// 音频暂停逻辑
		if (this.isPaused) {
			// 暂停倒计时 → 暂停音乐
			innerAudioContext.pause();
		} else {
			// 继续倒计时 → 继续播放音乐
			innerAudioContext.play();
		}
    },

    // 计算结束时间（时分）
    calcEndTime() {
      let now = new Date();
      let end = new Date(now.getTime() + this.totalSeconds * 1000);
      let hh = this.pad(end.getHours());
      let mm = this.pad(end.getMinutes());
      this.endTime = `${hh}:${mm}`;
    },

    // 秒数转时分秒
    secToTime(sec) {
      let h = this.pad(Math.floor(sec / 3600));
      let m = this.pad(Math.floor((sec % 3600) / 60));
      let s = this.pad(sec % 60);
      return `${h}:${m}:${s}`;
    },

    // 格式化显示
    formatTime(h, m, s) {
      return `${this.pad(h)}:${this.pad(m)}:${this.pad(s)}`;
    },

    // 补零
    pad(n) {
      return n < 10 ? "0" + n : n;
    },

    // 返回上一页
    goBack() {
      uni.navigateBack();
    },
	
	// 新增：播放音乐，循环
	playBGM() {
	      if (!this.bgAudioSrc) return
	      innerAudioContext.src = this.bgAudioSrc
	      innerAudioContext.loop = true  // 无限循环
	      innerAudioContext.play()
	},
	
	// 新增：结束播放音乐，彻底结束
	stopBGM() {
	      innerAudioContext.stop();
	},


  },
};
</script>

<style scoped>
/* 页面容器 */
.countdown-page {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #fff;
}

/* ==================== 顶部：一半高度、居中圆形 ==================== */
.top-section {
  flex: 1;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding-top: 80rpx;
}

.circle-box {
  width: 500rpx;
  height: 500rpx;
  border-radius: 50%;
  border: 8rpx solid #007aff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 30rpx;
  
}

/* 圆形上部：标题 */
.circle-title {
  font-size: 36rpx;
  color: #333;
}

/* 圆形中部：倒计时 */
.circle-time {
  font-size: 52rpx;
  font-weight: bold;
  color: #007aff;
}

/* 圆形底部：结束时间 */
.circle-end {
  font-size: 28rpx;
  color: #666;
}

/* ==================== 底部：左右按钮 ==================== */
.bottom-section {
/* 1. 开启固定定位 */
  position: fixed; 
  /* 2. 距离底部 100rpx (这就是你要的效果) */
  bottom: 100rpx; 
  /* 3. 通常固定定位的元素需要手动设置宽度，否则可能塌陷或占满 */
  width: 100%; 
  box-sizing: border-box; /* 确保 padding 不会撑大宽度导致溢出 */
  height: 200rpx;
  display: flex;
  align-items: center;
  justify-content: space-around;
  padding: 0 100rpx;
}

/* 圆形按钮 */
.btn-round {
  width: 100rpx;
  height: 100rpx;
  border-radius: 50%;
  background-color: #f5f5f5;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40rpx;
  color: #333;
}

/* 关闭 × */
.close-icon {
  font-size: 50rpx;
  color: #ff3b30;
}

/* 播放 || */
.play-icon {
  font-size: 40rpx;
  color: #007aff;
  letter-spacing: 4rpx;
  font-weight: bold;
}

/* 暂停 ▶ */
.pause-icon {
  font-size: 100rpx;
  color: #007aff;
  margin-left: 8rpx;
}
</style>