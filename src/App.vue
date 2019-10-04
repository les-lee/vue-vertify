<template>
  <div id="app">
    <button @click="alterImg">替换图片</button>
    <PtVertify
      ref="ptVertify"
      :options="{
        src: 'https://w.wallhaven.cc/full/96/wallhaven-96g2p8.jpg',
        //src: 'https://w.wallhaven.cc/full/39/wallhaven-392lk9.jpg',
        width: 300, // 整个组件的宽度 不能用百分比, 你可以用百分比计算好宽度后传进
        canvasHeight: 150, // 拼图背景canvas的高
        markWidth: 60, // 移动拼图的宽
        markHeight: 60, // 移动拼图的高
        slideWidth: 50, // 滑块的宽度
        gap: 20, // 移动拼图块与与随机截取区域的间隔, 预防随机截取区域过大导致 移动拼图块与截取的拼图快位置重叠
        r: 8, // 拼图圆弧的半径
        vertifyCallback: handleVertify, // 成功回调, 应与服务器进行交互验证, 返回一个promise, 必传
        vertifyErrorCallback: handleVertifyError // 验证失败的回调, 默认行为滑块会自动回到初始位置
      }"
    />
  </div>
</template>

<script>
import PtVertify from "./components/PtVertify";

export default {
  name: "app",
  components: {
    PtVertify
  },
  methods: {
    handleVertify() {
      console.log("验证成功");
      return new Promise((resolve, reject) => {
        resolve();
      });
    },
    handleVertifyError() {
      console.log("验证失败");
    },
    alterImg() {
      this.$refs.ptVertify.alternativeImgSrc(
        "http://a.hiphotos.baidu.com/image/pic/item/f603918fa0ec08fa3139e00153ee3d6d55fbda5f.jpg"
      );
    }
  }
};
</script>
