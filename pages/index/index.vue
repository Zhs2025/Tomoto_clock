<template>
	<view class="content">
		<image class="logo" src="/pages/index/static/tomoto_clock.jpg"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
		<view class="button-area">
			<button @click="button_click" class="button">点击以开始</button>
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
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
		margin-bottom: 50rpx;
	}
	
	.botton-area {
		display: flex;
		justify-content: center;
	}
	
	.button {
		height: 70rpx;
		width: 170rpx;
		color: #8f8f94;
		font-size: 24rpx;
	}
</style>
