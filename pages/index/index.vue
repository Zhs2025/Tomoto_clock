<template>
	<view class="content">
		<image class="logo" src="/pages/index/static/tomoto_clock.jpg"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
		<view class="button-area">
			<button @click="button_click" class="button">开始</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '欢迎使用'
			};
		},
		onLoad() {

		},
		methods: {
			button_click(){
				console.log('准备跳转...');				
				// 3. 调用 uni.navigateTo 进行跳转
				uni.navigateTo({
					url: '/pages/timer/timer', // 关键：路径必须与 pages.json 中配置的一致，且以 / 开头
					success: () => {
						console.log('跳转成功');
					},
					fail: (err) => {
						console.log('跳转失败', err);
						// 常见错误：如果路径写错，或者目标页面没在 pages.json 注册，这里会报错
					}
				});
			}
		}
	}
</script>

<style scoped>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		min-height: 100vh;  /* ← 加这行 */
		box-sizing: border-box;  /* ← 加这行，防止padding撑出屏幕 */
		height: 100vh;        /* 固定高度 = 屏幕高度，不被内容撑大 */
		overflow: hidden;     /* 超出部分直接隐藏，最关键 */
		
		
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
		/* 关键：变成圆形 */
		  border-radius: 25%;
		  
		  /* 可选：让圆形更美观，加个白色边框 */
		  border: 4rpx solid #ffffff;
		  
		  /* 防止图片变形 */
		  object-fit: cover;
	}

	.text-area {
		display: flex;
		justify-content: center;
		margin-bottom: 60rpx;
	}

	.title {
		font-size: 38rpx;
		color: #007aff;
		margin-bottom: 50rpx;
	}
	
	.botton-area {
		display: flex;
		justify-content: center;
		
	}
	
	.button {
		height: 70rpx;
		width: 170rpx;
		
		font-size: 28rpx;
		/* 字体加粗 */
		font-weight: bold;
		background-color:#007aff;
		
		/* 按钮边框 + 样式 */
		  color: #ffffff;
		  border: 2rpx solid #007aff; /* 边框 */
		  border-radius: 24rpx; /* 圆角 */
		  
		  text-align: center;
		  line-height: 70rpx; /* 垂直居中 */
		  
		  /* 🔥 核心：字体完全居中 */
		    display: flex;
		    align-items: center;
		    justify-content: center;
		    text-align: center;
	}
</style>
