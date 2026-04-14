<template>
  <!-- 最外层父容器 -->
  <view class="parent-box">
    <!-- 一级子容器：靠顶部 -->
    <view class="top-wrap">
      <!-- 二级子容器1：标题 -->
      <view class="title">背景音乐</view>

      <!-- 二级子容器2：卡片列表（固定2个） -->
      <view class="list-card">
        <!-- 选项1 -->
        <view class="item" @click="selectItem(0)">
          <text class="name">{{ musicList[0].name }}</text>
          <view :class="['circle', currentMusicIndex === 0 ? 'active' : '']"></view>
        </view>
        
        <!-- 选项2 -->
        <view class="item" @click="selectItem(1)">
          <text class="name">{{ musicList[1].name }}</text>
          <view :class="['circle', currentMusicIndex === 1 ? 'active' : '']"></view>
        </view>
        
      </view>
    </view>
	<!-- 一级容器： 靠下部-->
	<view class="second-section">
		<!-- 左二级容器：返回但不回传 -->
		<view class="second-left" @click="goBackWithoutData">
			取消
		</view>
		<!-- 右二级容器：点击返回上一页 + 回传数据 -->
		<view class="second-right" @click="goBackWithData">
			完成
		</view>
	</view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      // 背景音乐列表（固定2个）
      musicList: [
        { name: "小溪流水" },
        { name: "雨夜" }
      ],
      //保留数据 或 默认选中第一个（下标0的元素）
      currentMusicIndex: uni.getStorageSync('selectedMusicIndex') || 0
    };
  },
	methods: {
		// 点击切换选中
		selectItem(index) {
			
			// 1. 保存选中的项目
			this.currentMusicIndex = index;

			// 2. 保存音乐列表和音乐下标到本地存储，实现永久保存
			uni.setStorageSync('musicList', this.musicList);
			uni.setStorageSync('selectedMusicIndex', index);

			/* // 
				注意： 这种方法需要点击，存储在本地的数据不能更新
					所以，会导致，bgm的音乐和addtimer的音乐不一致，因为bgm当前显示的未被点击
			    功能： 发送全局事件，在addtimer中同步修改
				作用： 目前可有可无，暂时保留
			*/
			uni.$emit('musicChange', {
				musicIndex: index,
				currentMusic: this.musicList[index].name
			});
		},
		// 直接返回按键
		goBackWithoutData(){
			uni.navigateTo({
			  url: `/pages/addtimer/addtimer`,
			});
		},
		// 带着数据返回按键
		goBackWithData() {
			// 获取上一页实例，下两行
			const pages = getCurrentPages();
			const prevPage = pages[pages.length - 2]; 
			
			/* 
			   经过实验证明，使用prevPage.setData方法，是不行的，只能使用下面的方法
			   而应该 直接通过实例修改当前选中的音乐信息
			*/
			prevPage.$vm.musicIndex = this.currentMusicIndex;
			prevPage.$vm.currentMusic = this.musicList[this.currentMusicIndex].name;
			
			// 返回上一页
			uni.navigateBack();
		}
	}
};
</script>

<style scoped>
/* 父容器：全屏 */
.parent-box {
  width: 100%;
  min-height: 100vh;
  padding: 20rpx;
  box-sizing: border-box;
  background: #f5f5f5;
}

/* 一级子容器：靠顶部 */
.top-wrap {
  width: 100%;
  background: #fff;
  border-radius: 20rpx;
  padding: 30rpx;
  /* 防止padding撑出屏幕 */
  box-sizing: border-box;  
  box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08);
  margin-bottom: 20rpx;
}

/* 二级子容器1：标题 */
.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 30rpx;
}

/* 二级子容器2：卡片列表 */
.list-card {
  width: 100%;
  
}

/* 列表项：左右布局 */
.item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 24rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}

/* 最后一项去掉下划线 */
.item:last-child {
  border-bottom: none;
}

/* 名称样式 */
.name {
  font-size: 34rpx;
  color: #333;
}

/* 右侧圆圈默认样式 */
.circle {
  width: 32rpx;
  height: 32rpx;
  border-radius: 50%;
  border: 3rpx solid #ccc;
  box-sizing: border-box;
  transition: all 0.2s;
}

/* 选中高亮样式 */
.circle.active {
  border-color: #007aff; /* 统一高亮颜色 */
  background-color: rgba(7, 193, 96, 0.1);
}
/* 字体样式 */
.second-left {
  font-size: 34rpx;
  padding: 0 10rpx;
  color: #ffffff;
  
  /* 按钮盒子样式 */
  background-color: #007aff; 
  padding: 20rpx 60rpx;
  border-radius: 50rpx; /* 圆角 */
  box-shadow: 0 6rpx 16rpx rgba(255, 59, 48, 0.3); /* 阴影更高级 */
  
}
.second-right {
  font-size: 34rpx;
  padding: 0 10rpx;
  color: #ffffff;
  
  /* 按钮盒子样式 */
  background-color: #007aff; 
  padding: 20rpx 60rpx;
  border-radius: 50rpx; /* 圆角 */
  box-shadow: 0 6rpx 16rpx rgba(255, 59, 48, 0.3); /* 阴影更高级 */
}

/* 第二个一级子容器 */
.second-section {
  height: 90rpx;
  padding: 0 30rpx;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1rpx solid #eee;
  margin-bottom: 20rpx;
}
</style>