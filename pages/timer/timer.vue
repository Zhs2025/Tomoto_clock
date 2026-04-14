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

		<!-- 下方内容区域 -->
		<view class="content-scroll">
		<!-- 下方内容区域 -->
			<scroll-view 
				class="content" 
				scroll-y
				:scroll-top="scrollTop"    
				ref="scrollView"          
			>
			<!-- 循环显示列表项（二级子容器） -->
				<view 
					class="item" 
					:class="{active: activeIndex === index}"
					v-for="(item, index) in list" 
					:key="item.id"
					@click="setActive(index)"
					@longpress="enterEditMode"
				>
					<!-- 左三级子容器 -->
					<view class="item-left">{{ item.title }}</view>
					<!-- 右三级子容器 -->
					<view class="item-right">{{ item.time }}</view>
					<!-- 选择框 -->
					<view 
						v-if="editMode"
						class="check-box"
						:class="{checked:item.checked}"
					>
						<text v-if="item.checked">✔</text>
					</view>
				</view>
			</scroll-view>
		</view>
		
		<!-- 最底部悬浮按钮区域 -->
		<!-- 不使用 float-bottom 样式，后面的样式都是绝对位置 -->
		<view class="float-bottom" @click="bottomAction">
			<!-- 新增：编辑按钮（只有 editMode 为 true 时才显示） -->
			  <text 
			    class="edit-text" 
			    @click="editTextAction"
			    v-if="editMode"  
			  >
			    编辑
			  </text>
		    <image 
				v-if="!editMode"
				class="float-img" 
				src="/pages/timer/static/R-C.jpg"  
				mode="widthFix"
			>
			</image>
			<!-- 删除模式 -->
			<text v-else class="delete-text">删除</text>
			
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
				 // 读取本地存储，没有则使用默认值
				list: uni.getStorageSync('timeList') 
					? JSON.parse(uni.getStorageSync('timeList')) 
					: [
						{ id: 1001, title: "读书", time: "01:00:00", checked: false, music: 0  },
						{ id: 1002, title: "绘画", time: "02:00:00", checked: false, music: 0  },
						{ id: 1003, title: "编程", time: "03:00:00", checked: false, music: 0  },
					  ],
				// 记录当前点击的索引
				activeIndex: -1, 
				// 是否进入删除模式
				editMode: false,
				// 控制滚动位置
				scrollTop: 0,  
				// 新增：当前音乐
				currentMusic: 0,
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
				const musicList = uni.getStorageSync('musicList')|| [
					{ name: "小溪流水" },
					{ name: "雨夜" }
				  ];
				console.log(musicList);
				const currentMusicIndex = uni.getStorageSync('selectedMusicIndex') || 0;
				console.log(currentMusicIndex);
				const currentMusicName = musicList[currentMusicIndex].name;

				uni.navigateTo({
					url: '/pages/addtimer/addtimer'  
				});
				console.log("当前播放音乐：", currentMusicName);
			},
			
			// 底部按钮点击
		    bottomAction() {
			
			    // 普通模式
			    if (!this.editMode) {
			      this.goBottomPage()
			      return
			    }
			
			    // 删除选中
			    this.list = this.list.filter(item => !item.checked)
				
				// 保存删除
				this.saveList();
			    // 退出编辑模式
			    this.editMode = false
			  },
			  
			// 点击底部图片 跳转到计时页面
			goBottomPage() {
			   
			    // 转数字 并 格式化时间为 00:00:00 格式
			    const h = Number(this.hour) || 0
			    const m = Number(this.minute) || 0
			    const s = Number(this.second) || 0
			    const selectedTime = `${h}:${m}:${s}`
				
			    // 获取标题, 先更新 后读入
			    let inputText = ""
			    if (this.activeIndex !== -1 && this.list[this.activeIndex]) {
			        inputText = this.list[this.activeIndex].title
			    }
			
				// 获取音乐编号, 先更新 后读入
				this.currentMusic = this.list[this.activeIndex].music;
				const music = Number(this.currentMusic) || 0
							
			    // 跳转
			    uni.navigateTo({
			        url: `/pages/timing/timing?selectedTime=${encodeURIComponent(selectedTime)}&inputText=${encodeURIComponent(inputText)}&music=${music}`
			    })
			},
			
			// 点击列表元素设置激活项
			setActive(index) {
				
			  // 设置高亮项
			  this.activeIndex = index;
			  
			  // 移动选择时间盒 到 当前时间
			  let item = this.list[index];
			  let time = item.time;
			  let [h, m, s] = time.split(":");
			  this.setWheelTo("H", "hour", h);
			  this.setWheelTo("M", "minute", m);
			  this.setWheelTo("S", "second", s);
			  
			  //编辑模式选择
			  if (this.editMode) {
				this.list[index].checked = !this.list[index].checked
			  }
			},
			
			setWheelTo(type, fulltype, value) {
			  value = Number(value);
			  let list = this[`${fulltype}List`];
			  let idx = list.indexOf(value);
			  // 计算一次全列表元素长度，用于初始显示到第二次循环
			  let smoothShow = list.length / 4;
			  idx += smoothShow;
			  this[`currentIndex${type}`] = idx;
			  this[`scrollTop${type}`] = idx  * this.itemHeight;
			  this[type.toLowerCase()] = value;
			},
			
			// 长按进入编辑模式
			enterEditMode() {
				this.editMode = true
			},
			
			// 保存到本地
			saveList() {
			  uni.setStorageSync("timeList", JSON.stringify(this.list));
			},
			// 新增：滚动到列表最底部
			scrollToBottom() {
			  // 延迟一小帧，确保列表已渲染完成
			  setTimeout(() => {
			    this.scrollTop = 999999; // 直接设一个超大数，自动到底部
			  }, 10);
			},
			editTextAction() {
				uni.navigateTo({
					url: '/pages/addtimer/addtimer'  
				});
			}
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
	  height: 100vh;        /* 固定高度 = 屏幕高度，不被内容撑大 */
	  overflow: hidden;     /* 超出部分直接隐藏，最关键 */
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
	.result {
	  margin-top: 60rpx;
	  height: 60rpx;       /* 高度 = 父容器高度 */
	  font-size: 30rpx;
	  font-style: italic;
	  color: #007aff;
	  margin-bottom: 30rpx;
	}
	/* 中部标题栏 */
	.midheader {
	  margin-top: 60rpx;
	  width: 100%;
	  height: 60rpx;
	  padding: 0 30rpx;
	  margin-bottom: 20rpx;
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
	.content-scroll {
		margin-top: 0rpx;
		display: flex;
		width: 100%;    
		height: 640rpx; 
	}
	/* 下方内容区域（填满剩余高度） */
	.content {
	  flex: 1;
	  width: 100%;
	  padding: 0 rpx;
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
	
	.check-box{
	  width:40rpx;
	  height:40rpx;
	  border:2rpx solid #ccc;
	  border-radius:6rpx;
	  display:flex;
	  align-items:center;
	  justify-content:center;
	}
	
	.check-box.checked{
	  background:#007aff;
	  color:#fff;
	  border-color:#007aff;
	}
	
	
	/* 图片 */
	.float-img {
	    width: 150rpx;
		height: 150rpx;
		/* 下面是固定定位关键代码 */
		position: fixed;    /* 固定定位，脱离文档流 */
		bottom: 150rpx;     /* 距离底部 150rpx（你原来的 margin-bottom） */
		left: 50%;          /* 水平居中 */
		transform: translateX(-50%); /* 水平居中 */
		z-index: 999;       /* 保证在最上层，不被盖住 */
	}
	/* 删除按钮 */
	.edit-text{
		color:#ffffff;
		font-size:38rpx;
	
		/* 按钮盒子样式 */
		background-color: #007aff; 
		padding: 20rpx 60rpx;
		border-radius: 50rpx; /* 圆角 */
		box-shadow: 0 6rpx 16rpx rgba(255, 59, 48, 0.3); /* 阴影更高级 */
	
	
		position: fixed;    /* 固定定位，脱离文档流 */
		bottom: 150rpx;     /* 距离底部 150rpx（你原来的 margin-bottom） */
		left: 100rpx;
	}
	/* 删除按钮 */
	.delete-text{
		color:#ffffff;
		font-size:38rpx;

		/* 按钮盒子样式 */
		background-color: #007aff; 
		padding: 20rpx 60rpx;
		border-radius: 50rpx; /* 圆角 */
		box-shadow: 0 6rpx 16rpx rgba(255, 59, 48, 0.3); /* 阴影更高级 */


		position: fixed;    /* 固定定位，脱离文档流 */
		bottom: 150rpx;     /* 距离底部 150rpx（你原来的 margin-bottom） */
		right: 100rpx;
	}
</style>

