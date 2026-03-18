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
      // 新增：保留数据 或 默认选中第一个（下标0）
      currentMusicIndex: uni.getStorageSync('selectedMusicIndex') || 0
    };
  },
  methods: {
    // 点击切换选中
    selectItem(index) {
      this.currentMusicIndex = index;
	  
	  // 2. 保存音乐列表和音乐下标到本地存储（永久保存）
	  uni.setStorageSync('musicList', this.musicList);
	  uni.setStorageSync('selectedMusicIndex', index);
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
</style>