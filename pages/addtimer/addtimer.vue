<template>
	<!-- 总父容器 -->
	<view class="page-container">
		<!-- 上部一级容器 -->
		<view class="top-section">
			<!-- 左二级容器 -->
			<view class="top-left" @click="goBackWithData">
				取消
			</view>
			<!-- 右二级容器：点击返回上一页 + 回传数据 -->
			<view class="top-right" @click="goBackWithData">
				完成
			</view>
		</view>
		
		<!-- 中部一级容器：时分秒轮盘 -->
		<view class="middle-section">
		  <view class="time-picker">
			<!-- 时 -->
			<view class="picker-box">
			  <scroll-view scroll-y :scroll-top="scrollTopH" @scroll="onScrollH" @touchend="onEndH" class="picker-scroll">
				<view class="picker-list">
				  <view v-for="(num, idx) in hourList" :key="idx" class="picker-item">
					{{ num }}
				  </view>
				</view>
			  </scroll-view>
			</view>
			<!-- 分 -->
			<view class="picker-box">
			  <scroll-view scroll-y :scroll-top="scrollTopM" @scroll="onScrollM" @touchend="onEndM" class="picker-scroll">
				<view class="picker-list">
				  <view v-for="(num, idx) in minuteList" :key="idx" class="picker-item">
					{{ num }}
				  </view>
				</view>
			  </scroll-view>
			</view>
			<!-- 秒 -->
			<view class="picker-box">
			  <scroll-view scroll-y :scroll-top="scrollTopS" @scroll="onScrollS" @touchend="onEndS" class="picker-scroll">
				<view class="picker-list">
				  <view v-for="(num, idx) in secondList" :key="idx" class="picker-item">
					{{ num }}
				  </view>
				</view>
			  </scroll-view>
			</view>
		  </view>
		</view>

		<!-- 下部一级容器 -->
		<view class="bottom-section">
			<!-- 下-上二级容器：输入框 + 点击下边框高亮 -->
			<view 
				class="input-wrapper" 
				:class="{active: inputActive}" 
				@click="focusInput"
			>
				<input 
					v-model="inputText" 
					class="input"
					placeholder="专注任务名称" 
					@blur="inputActive=false" 
					confirm-type="done"
				/>
					<!-- 【新增】清除按钮：只有当 inputText 有值时显示 -->
					<!-- @click.stop 阻止事件冒泡，防止触发外层的 focusInput -->
					<view 
						v-if="inputText" 
						class="clear-btn" 
						@click.stop="clearInput"
					>
						✕ <!-- 或者使用图标组件 <uni-icons type="closeempty" ... /> -->
					</view>
			</view>
			
			<!-- 下-下二级容器：点击跳转页面 -->
			<view class="jump-item" @click="goToOtherPage">
				背景音乐
			</view>
		</view>
    </view>
</template>

<script>
export default {
  data() {
    return {
      // 轮盘基础配置
      itemHeight: 60,
      // 时
      hourList: [],
      currentH: 0,
      scrollTopH: 0,
      tmpH: 0,
      // 分
      minuteList: [],
      currentM: 0,
      scrollTopM: 0,
      tmpM: 0,
      // 秒
      secondList: [],
      currentS: 0,
      scrollTopS: 0,
      tmpS: 0,

      // 输入框
      inputText: "",
      inputActive: false,
    };
  },
  onLoad() {
    this.initWheel("hour", 0, 23);
    this.initWheel("minute", 0, 59);
    this.initWheel("second", 0, 59);
  },
  methods: {
    // ========== 初始化纯循环轮盘（单列表、无高亮） ==========
    initWheel(type, min, max) {
      let list = [];
      for (let i = min; i <= max; i++) list.push(i);
      this[`${type}List`] = list;
      this[`scrollTop${type.toUpperCase()}`] = 0;
    },

    // ========== 滚动监听 ==========
    onScrollH(e) { this.tmpH = e.detail.scrollTop; },
    onScrollM(e) { this.tmpM = e.detail.scrollTop; },
    onScrollS(e) { this.tmpS = e.detail.scrollTop; },

    // ========== 滚动结束（纯循环） ==========
    onEndH() { this.calc("H", "hour", this.tmpH); },
    onEndM() { this.calc("M", "minute", this.tmpM); },
    onEndS() { this.calc("S", "second", this.tmpS); },

    calc(type, fulltype, top) {
      const list = this[`${fulltype}List`];
      const len = list.length;
      let idx = Math.round(top / this.itemHeight);
      let loopIdx = ((idx % len) + len) % len;
      this[`current${type}`] = list[loopIdx];
      this[`scrollTop${type}`] = loopIdx * this.itemHeight;
    },

    // ========== 输入框点击：下边框高亮 ==========
    focusInput() {
      this.inputActive = true;
    },

	clearInput() {
		this.inputText = '';
		// 清空后通常希望保持聚焦状态，方便继续输入
		this.inputActive = true; 

		// 可选：如果需要在清空后立即再次聚焦输入框（防止键盘收起）
		// 这里可能需要使用 nextTick 或定时器，视具体框架表现而定
		// setTimeout(() => { this.inputActive = true; }, 10);
	},
	
    // ========== 点击右上角：返回上一页 + 传参 ==========
    goBackWithData() {
      uni.navigateBack({
        delta: 1,
        success: () => {
          const time = `${this.currentH}:${this.currentM}:${this.currentS}`;
          const text = this.inputText;
          
          // 回传数据到上一页
          const pages = getCurrentPages();
          const prevPage = pages[pages.length - 2];
          prevPage.setData({
            selectedTime: time,
            inputText: text
          });
        }
      });
    },

    // ========== 底部点击：跳转到其他页面 ==========
    goToOtherPage() {
      uni.navigateTo({
        url: "/pages/other/other"
      });
    }
  }
};
</script>

<style scoped>
/* 总容器 */
.page-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #eeeeee;
}
/* 上部  */
.top-section {
  height: 90rpx;
  padding: 0 30rpx;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1rpx solid #eee;
  margin-bottom: 20rpx;
}
.top-left {
  font-size: 34rpx;
  padding: 0 10rpx;
  color: #007aff;
  
}
.top-right {
  font-size: 34rpx;
  padding: 0 10rpx;
  color: #007aff;
}

/* ====================== 中部：轮盘 ====================== */
.middle-section {
  display: flex;
  align-items: flex-start; 
  justify-content: center;
  background-color: #ffffff;
  margin-bottom: 40rpx;
  width: 80%;
   margin-left: auto;
     margin-right: auto;
   
     /* 【可选美化】增加圆角和阴影，让它更像一张独立的卡片 */
     border-radius: 20rpx;
     box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
     
     /* 【可选】内部内边距，防止内容贴到白色边缘 */
     padding: 30rpx; 
}
.time-picker {
  display: flex;
  width: 70%;
  height: 360rpx;
}
.picker-box {
  flex: 1;
  height: 100%;
  overflow: hidden;
}
.picker-scroll {
  height: 100%;
}
.picker-list {
  padding: 170rpx 0;
}
.picker-item {
  height: 60rpx;
  line-height: 60rpx;
  text-align: center;
  font-size: 32rpx;
  color: #333;
}

/* 下部 */
.bottom-section {
  height: 160rpx;
  width: 80%; /* 宽度必须和中部完全一致 */
  margin-left: auto;
  margin-right: auto;
  padding: 20rpx 30rpx;
  
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  background-color: #ffffff;
  
  /* 圆角 */
  border-top-left-radius: 20rpx; 
  border-top-right-radius: 20rpx;
  border-bottom-left-radius: 20rpx;
  border-bottom-right-radius: 20rpx;
  
  /* 加回分割线 */
  border-top: 1rpx solid #eee; 
  
  /* 阴影只加在整体最下方，或者不加，让整体更扁平 */
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
}

/* 输入框 + 点击下划线高亮 */
.input-wrapper {
  position: relative; /* 【关键】作为定位父级 */
  height: 80rpx;
  border-bottom: 2rpx solid #f9f9f9;
  display: flex;
  align-items: center;
}
.input-wrapper.active {
  border-bottom: 2rpx solid #007aff;
}
.input {
  width: 100%;
  font-size: 34rpx;
  z-index: 1; /* 输入框层级设为 1 */
  
}
/* 【新增】清除按钮样式 */
.clear-btn {
  position: absolute;
  right: 20rpx; /* 距离右边的距离 */
  top: 50%;
  transform: translateY(-50%); /* 垂直居中 */
  z-index: 10; /* 【关键】清除按钮层级必须高于输入框 */
  
  width: 40rpx;
  height: 40rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  
  font-size: 36rpx; /* 叉号大小 */
  color: #ffffff; /* 白色 */
  background-color: #7a7a7a; /* 深灰背景圆 */
  border-radius: 50%;
  
  /* 点击态反馈 */
  active-opacity: 0.6; 
}
/* 点击跳转项 */
.jump-item {
  height: 80rpx;
  line-height: 80rpx;
  font-size: 34rpx;
  color: #333;
}
</style>