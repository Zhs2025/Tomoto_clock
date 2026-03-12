<template>
	<view class="container">
		<!-- 竖向数字轮盘主体 -->
		<view class="time-picker">
			<view class="picker-box">
				<!-- 选中高亮遮罩 -->
				<view class="picker-highlight"></view>
				<!-- 可滑动的数字列表 -->
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
						    :class="{ active: index === currentIndexH }"
						>
						    {{ num }}
						</view>
					</view>
				</scroll-view>
			</view>
			<view class="picker-box">
				<!-- 选中高亮遮罩 -->
				<view class="picker-highlight"></view>
				<!-- 可滑动的数字列表 -->
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
						    :class="{ active: index === currentIndexM }"
						>
						    {{ num }}
						</view>
					</view>
				</scroll-view>
			</view>
			<view class="picker-box">
				<!-- 选中高亮遮罩 -->
				<view class="picker-highlight"></view>
				<!-- 可滑动的数字列表 -->
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
						    :class="{ active: index === currentIndexS }"
						>
						    {{ num }}
						</view>
					</view>
				</scroll-view>
			</view>
		</view>
		
		<!-- 当前选中时间 -->
		<view class="result">当前选择：{{ hour }} : {{ minute }} : {{ second }}</view>
		
		<!-- 中部标题栏 -->
		<view class="midheader">
		      <!-- 左侧标题 -->
		      <view class="midheader-left">常用专注时间</view>
		      <!-- 右侧按钮：点击跳转 -->
		      <view class="midheader-right" @click="goAddPage">
				添加
		      </view>
		</view>

		<!-- 下方内容区域（可滚动、占满剩余高度） -->
		<scroll-view class="content" scroll-y>
			<!-- 循环显示列表项（二级子容器） -->
			<view 
				class="item" 
				:class="{active: activeIndex === index}"
				v-for="(item, index) in list" 
				:key="index"
				@click="setActive(index)"
			>
				<!-- 左三级子容器 -->
				<view class="item-left">{{ item.title }}</view>
				<!-- 右三级子容器 -->
				<view class="item-right">{{ item.time }}</view>
			</view>
		</scroll-view>
	
		<!-- 最底部悬浮按钮（最高层级、可点击、显示图片）  -->
		<view class="float-bottom" @click="goBottomPage">
		    <image class="float-img" src="/pages/timer/static/R-C.jpg"  mode="widthFix"></image>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// 时间轮盘初始配置
				// 每一项高度（必须与CSS一致）
			    itemHeight: 120, 
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
				// 时间列表初始配置
				list: [
					{ title: "读书", time: "01:00:00" },
					{ title: "绘画", time: "02:00:00" },
					{ title: "编程", time: "03:00:00" },
				],
				// 新增：记录当前点击的索引
				activeIndex: -1, 
			};
		},
		onLoad() {
			// 1. 定义 CSS 中的 rpx 值
			const itemHeightRpx = 120;     
			// 2. 转换为 px (这是 scroll-top 需要的单位)
			this.itemHeight = uni.upx2px(itemHeightRpx);
			
			this.initList("h", "hour", 0, 23, 0);
			this.initList("m", "minute", 0, 59, 15);
			this.initList("s", "second", 0, 59, 0);
		},
		methods: {
			// 初始化循环列表
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
				
				console.log(`currentIndex${type.toUpperCase()}`);
				
				this[`scrollTop${type.toUpperCase()}`] = (mid + defaultnumber) * this.itemHeight;
				
				console.log(`scrollTop${type.toUpperCase()}`);
				
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
			
			handleEnd(type, fulltype, top) {
				let idx = Math.round(top / this.itemHeight);
				let list = this[`${fulltype}List`];
				idx = Math.max(0, Math.min(idx, list.length - 1));
				this[`currentIndex${type.toUpperCase()}`] = idx;
				this[`scrollTop${type.toUpperCase()}`] = idx * this.itemHeight;
				let realVal = list[idx];
				this[fulltype] = realVal;
			},
			
			// 添加键跳转，添加时间列表元素
			goAddPage() {
				uni.navigateTo({
					url: '/pages/addtimer/addtimer'  // 改成你自己的添加页面路径
				});
			},
			
			// 点击底部悬浮图片 → 跳转到新页面
			goBottomPage() {
				uni.navigateTo({
					url: '/pages/timing/timing'  // 改成你自己的底部跳转页面
				});
			},
			
			// 点击列表元素设置激活项
			setActive(index) {
			  this.activeIndex = index;
			  let item = this.list[index];
			  let time = item.time;
			  let [h, m, s] = time.split(":");
			  
			  this.setWheelTo("H", "hour", h);
			  this.setWheelTo("M", "minute", m);
			  this.setWheelTo("S", "second", s);
			},
			
			setWheelTo(type, fulltype, value) {
			  value = Number(value);
			  let list = this[`${fulltype}List`];
			  let idx = list.indexOf(value);
			  let smoothShow = list.length / 4;
			  idx += smoothShow;
			  this[`currentIndex${type}`] = idx;
			  this[`scrollTop${type}`] = idx  * this.itemHeight;
			  this[type.toLowerCase()] = value;
			},
		},
	};
</script>

<style scoped>
	/* 页面容器 */
	.container {
	  padding: 40rpx;
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	  background-color: #eeeeee;
	  min-height: 100vh;  /* ← 加这行 */
	  box-sizing: border-box;  /* ← 加这行，防止padding撑出屏幕 */
	}
	/* 时间轮盘 */
	.time-picker {
		display: flex;
		width: 100%;    /* 父容器宽度铺满 */
		height: 360rpx; /* 统一高度，三个子元素都会继承 */
	}
	/* 轮盘外框 */
	.picker-box {
	  flex: 1;
	  height: 100%;
	  position: relative;
	  overflow: hidden;
	  border: 1rpx solid #eee;
	  border-radius: 12rpx;
	  margin: 0 30rpx;
	}
	/* 选中高亮条 */
	.picker-highlight {
	  position: absolute;
	  top: 50%;
	  left: 0;
	  width: 100%;
	  height: 120rpx;
	  transform: translateY(-50%);
	  background-color: #f5f5f5;
	  z-index: 0;
	  pointer-events: none;
	}
	/* 滚动区域 */
	.picker-scroll {
	  height: 100%;
	  width: 100%;
	}
	.picker-list {
	  padding: 120rpx 0; /* 上下留白，让第一项/最后一项能居中 */
	}
	/* 每一项样式 */
	.picker-item {
	  height: 120rpx;
	  line-height: 120rpx;
	  text-align: center;
	  font-size: 64rpx;
	  color: #999;
	  transition: all 0.2s;
	}
	/* 选中项样式 */
	.picker-item.active {
	  font-size: 90rpx;
	  color: #007aff;
	  font-weight: bold;
	}
	/* 结果显示区域 */
	.result-show {
	  display: flex;
	  width: 100%;    /* 父容器宽度铺满 */
	  height: 60rpx; /* 统一高度，三个子元素都会继承 */
	}
	/* 结果 */
	.result {
	  margin-top: 60rpx;
	  height: 100%;       /* 高度 = 父容器高度 */
	  font-size: 30rpx;
	  font-style: italic;
	  color: #007aff;
	}
	/* 上方标题栏 */
	.midheader {
	  margin-top: 60rpx;
	  width: 100%;
	  height: 60rpx;
	  padding: 0 30rpx;
	  
	  display: flex;
	  justify-content: space-between; /* 左右分开 */
	  align-items: center;
	  border-bottom: 1rpx solid #eee;
	  box-sizing: border-box;
	}
	
	.midheader-left {
	  font-size: 32rpx;
	  font-weight: bold;
	  color: #333;
	}
	
	.midheader-right {
	  font-size: 32rpx;
	  color: #007aff;
	}
	
	/* 下方内容区域（填满剩余高度） */
	.content {
	  flex: 1;
	  width: 100%;
	  padding: 0 25rpx;
	  box-sizing: border-box;
	  min-height: 0;  /* ← 加这行，修复flex子元素不收缩的bug */
	}
	
	/* 列表项（二级子容器） */
	.item {
	  width: 100%;
	  height: 100rpx;
	  background-color: #fff;
	  border: 1rpx solid #ddd; 
	  box-sizing: border-box; /* 重要！确保边框包含在 120rpx 高度内，否则总高度会变高 */
	  border-radius: 20rpx;
	  margin-bottom: 20rpx;
	  padding: 0 30rpx;
	  display: flex;
	  justify-content: space-between;
	  align-items: center;
	  box-sizing: border-box;
	}
	/* 列表激活 */
	.item.active {
	    background-color: #d9f3ff; /* 点击后的背景色 */
	    border-color: #007aff;     /* 点击后的边框色 */
	}

	/* 左三级子容器 */
	.item-left {
	  font-size: 38rpx;
	  color: #333;
	}
	
	/* 右三级子容器 */
	.item-right {
	  font-size: 38rpx;
	  color: #999;
	}
	
	/* 图片 */
	.float-img {
	  width: 150rpx;
	  height: 150rpx;
	  margin-bottom: 150rpx;
	}
</style>

