<template>
	<!-- 总父容器 -->
	<view class="page-container">
		<!-- 上部一级容器 -->
		<view class="top-section">
			<!-- 左二级容器 -->
			<view class="top-left" @click="goBackWithoutData">
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
			  <scroll-view 
			    scroll-y 
			    :scroll-top="scrollTopH" 
			    @scroll="onScrollH" 
			    class="picker-scroll"
			  >
			    <view class="picker-list">
			      <view 
			        v-for="(num, index) in hourList" 
			        :key="index" 
			        class="picker-item"
			      >
			        {{ num }}
			      </view>
			    </view>
			  </scroll-view>
			</view>
			
			<!-- 分 -->
			<view class="picker-box">
			  <scroll-view 
			    scroll-y 
			    :scroll-top="scrollTopM" 
			    @scroll="onScrollM" 
			    class="picker-scroll"
			  >
			    <view class="picker-list">
			      <view 
			        v-for="(num, index) in minuteList" 
			        :key="index" 
			        class="picker-item"
			      >
			        {{ num }}
			      </view>
			    </view>
			  </scroll-view>
			</view>
			
			<!-- 秒 -->
			<view class="picker-box">
			  <scroll-view 
			    scroll-y 
			    :scroll-top="scrollTopS" 
			    @scroll="onScrollS" 
			    class="picker-scroll"
			  >
			    <view class="picker-list">
			      <view 
			        v-for="(num, index) in secondList" 
			        :key="index" 
			        class="picker-item"
			      >
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
      itemHeight: 100,
	  // 计时器
	  _timerH: null,
	  _timerM: null,
	  _timerS: null,
      // 时
      hourList: [],
      currentIndexH: 0,
      hour: 0,
      scrollTopH: 0,
      startScrollTopH: 0,
      // 分
      minuteList: [],
      currentIndexM: 0,
      minute: 0,
      scrollTopM: 0,
      startScrollTopM: 0,
      // 秒
      secondList: [],
      currentIndexS: 0,
      second: 0,
      scrollTopS: 0,
      startScrollTopS: 0,

      // 输入框
      inputText: "",
      inputActive: false,
    };
  },
  onLoad() {
		// 1. 定义 CSS 中的 rpx 值
		const itemHeightRpx = 100;     
		// 2. 转换为 px (这是 scroll-top 需要的单位)
		this.itemHeight = uni.upx2px(itemHeightRpx);
		this.initList("h", "hour", 0, 23, 0);
		this.initList("m", "minute", 0, 59, 15);
		this.initList("s", "second", 0, 59, 0);
    },
  methods: {
    // ========== 初始化纯循环轮盘（单列表、无高亮） ==========
    initList(type, fulltype, min, max, defaultnumber) {
    	let list = [];
    	for (let i = min; i <= max; i++) list.push(i);
    	let len = list.length;
    	let loop = 4;
    	let fullList = [];
    	for (let i = 0; i < loop; i++) fullList = fullList.concat(list);
    	this[`${fulltype}List`] = fullList;
    	let mid = Math.floor(loop / 2) * len;
    	this[`currentIndex${type.toUpperCase()}`] = mid + defaultnumber;
    	//console.log(`currentIndex${type.toUpperCase()}`);
    	this[`scrollTop${type.toUpperCase()}`] = (mid + defaultnumber) * this.itemHeight;
    	//console.log(`scrollTop${type.toUpperCase()}`);
    	this[fulltype] = list[defaultnumber];
    	
    },

    // 滚动中, 获取当前滚动距离 (px)
    onScrollH(e) {
        this.startScrollTopH = e.detail.scrollTop;
        if (this._timerH) clearTimeout(this._timerH);
        this._timerH = setTimeout(() => {
            this.handleEnd("h", "hour", this.startScrollTopH);
        }, 150);
    },
    onScrollM(e) {
        this.startScrollTopM = e.detail.scrollTop;
        if (this._timerM) clearTimeout(this._timerM);
        this._timerM = setTimeout(() => {
            this.handleEnd("m", "minute", this.startScrollTopM);
        }, 150);
    },
    onScrollS(e) {
        this.startScrollTopS = e.detail.scrollTop;
        if (this._timerS) clearTimeout(this._timerS);
        this._timerS = setTimeout(() => {
            this.handleEnd("s", "second", this.startScrollTopS);
        }, 150);
    },

    // ========== 滚动结束（纯循环） ==========
    handleEnd(type, fulltype, top) {
    	let idx = Math.round(top / this.itemHeight);
    	let list = this[`${fulltype}List`];
    	idx = Math.max(0, Math.min(idx, list.length - 1));
    	this[`currentIndex${type.toUpperCase()}`] = idx;
    	this[`scrollTop${type.toUpperCase()}`] = idx * this.itemHeight;
    	let realVal = list[idx];
    	this[fulltype] = realVal;
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
	// 点击取消，直接返回上一页
	goBackWithoutData(){
		uni.navigateTo({
		  url: `/pages/timer/timer`,
		});
	},
    // ========== 点击右上角：返回上一页 + 传参 ==========
    goBackWithData() {
      const time = `${this.hour}:${this.minute}:${this.second}`;
      const text = this.inputText || '专注时间';
      console.log(time);
      console.log(text);
      // 【关键】在这里生成唯一 ID，不需要 timer 页面配合维护 nextId
      const newItem = {
        id: Date.now(), // 例如：1710324567890，绝对唯一
        title: text,
        time: time,
		checked: false
      };
       
      console.log(newItem.id);
    
      // ======================================
      // 【只修复这里：先拿上一页、先赋值，再返回】
      // ======================================
      const pages = getCurrentPages();
      const prevPage = pages[pages.length - 2]; // 👈 修复成 -2
      console.log(prevPage.route);
      console.log(prevPage.$vm.data);
	  console.log(prevPage.$vm);
      console.log(prevPage.$vm.list);
      
      if (prevPage) {
        const currentList = prevPage.$vm.list || [];
        // 直接合并
        prevPage.setData({
          list: [...currentList, newItem]
        });
		prevPage.$vm.list = [...currentList, newItem];
		const newIndex = prevPage.$vm.list.length - 1;
		prevPage.$vm.setActive(newIndex);
		// 保存
		prevPage.$vm.saveList(); 
		// 新增：滑到底
		prevPage.$vm.scrollToBottom() 
      }
      console.log(prevPage.$vm.list);
	  
    
      // 然后再执行返回
      uni.navigateBack({
        delta: 1
      });
    },
		
    // ========== 底部点击：跳转到其他页面 ==========
    goToOtherPage() {
      uni.navigateTo({
        url: "/pages/bgm/bgm"
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

/* 中部：轮盘  */
.middle-section {
  display: flex;
  align-items: flex-start; 
  justify-content: center;
  background-color: #ffffff;
  margin-bottom: 40rpx;
  width: 80%;
   margin-left: auto;
     margin-right: auto;
   
     /* 增加圆角和阴影，让它更像一张独立的卡片 */
     border-radius: 20rpx;
     box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
     
     /* 内部内边距，防止内容贴到白色边缘 */
     padding: 30rpx; 
}
.time-picker {
  display: flex;
  position: relative; /* 【关键】作为定位父级 */
  justify-content: space-around; /* 三个滚轮均匀分布 */
  width: 70%;
  height: 300rpx;
}
/* 【核心】绘制上下两条横线 */
.time-picker::before,
.time-picker::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  height: 1rpx; /* 线条粗细 */
  background-color: rgba(0, 0, 0, 0.1); /* 线条颜色，淡灰色 */
  z-index: 10; /* 确保线条在滚轮内容之上 */
  pointer-events: none; /* 【至关重要】让点击事件穿透线条，不影响滚轮滑动 */
}

/* 上线：距离顶部的位置 = (容器高度 - 选中项高度) / 2 */
/* 假设容器 400rpx, 选中项 80rpx -> (400-80)/2 = 160rpx */
.time-picker::before {
  top: 90rpx; 
}

/* 下线：距离底部的位置 = (容器高度 - 选中项高度) / 2 */
.time-picker::after {
  bottom: 110rpx;
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
  padding: 100rpx 0;
}
.picker-item {
  height: 100rpx;
  line-height: 60rpx;
  text-align: center;
  font-size: 40rpx;
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