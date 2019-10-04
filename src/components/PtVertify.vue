<template>
  <div :style="{width:options.width + 'px'}" id="ptVertify">
    <canvas ref="bg" id="vertify_main"></canvas>
    <canvas ref="mark" id="vertify_mark"></canvas>
    <div ref="slideWrap" class="slider-wrap">
      <span :class="['slided',isFail ? 'fail': '',isSuccess ? 'success':'']" ref="slided"></span>
      <span
        ref="slide"
        :class="['slide', isFail ? 'fail': '', isSuccess ? 'success':'']"
        @mousedown="mousedown($event)"
      >>></span>
      <span class="slider">滑动滑块完成验证</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "ptVertify",
  props: {
    options: {
      type: Object,
      default: function() {
        return {
          width: 300,
          canvasHeight: 150,
          vertifyErrorCallback: () => {},
          markWidth: 60,
          markHeight: 60,
          slideWidth: 50,
          gap: 20,
          r: 8
        };
      }
    }
  },
  components: {},
  data: function() {
    return {
      isDown: false,
      isFail: false,
      isSuccess: false,
      vertifyImgSrc: this.options.src
    };
  },
  positionMeta: {
    disX: 0,
    disY: 0,
    targetX: -1,
    slideLeft: 0
  },
  methods: {
    mousedown(e) {
      this.isDown = true;
      this.$options.positionMeta.disX = e.clientX - this.$refs.slide.offsetLeft;
      this.$options.positionMeta.disY = e.clientY - this.$refs.slide.offsetTop;
    },
    mousemove(e) {
      if (this.isDown) {
        this.$options.slideLeft = e.pageX - this.$options.positionMeta.disX;
        if (
          e.pageX - this.$options.positionMeta.disX <
          this.$refs.slideWrap.offsetLeft
        ) {
          this.$refs.slide.style.left = this.$refs.mark.style.left = this.$refs.slided.style.width =
            0 + "px";
          return;
        }
        if (
          e.pageX - this.$options.positionMeta.disX >
          this.$refs.slideWrap.clientWidth - this.options.slideWidth
        ) {
          this.$refs.slide.style.left = this.$refs.mark.style.left = this.$refs.slided.style.width =
            this.$refs.slideWrap.clientWidth - this.options.slideWidth + "px";
          return;
        }
        this.$refs.slide.style.left = this.$refs.mark.style.left = this.$refs.slided.style.width =
          this.$options.slideLeft + "px";
      }
    },
    mouseup() {
      if (!this.isDown) return;
      this.isDown = false;
      this.vertify();
    },
    getMarkPostion() {
      return {
        x:
          this.options.markWidth +
          this.options.gap +
          Math.floor(
            Math.random() *
              (this.options.width -
                this.options.markWidth * 2 -
                this.options.gap -
                this.options.r * 2 -
                2) +
              1
          ),
        y:
          this.options.r * 2 -
          2 +
          Math.floor(
            Math.random() *
              (this.options.canvasHeight -
                this.options.markHeight -
                (this.options.r * 2 - 2)) +
              1
          )
      };
    },
    alternativeImgSrc(imgSrc) {
      this.$refs.bg.width = this.$refs.mark.width = this.options.width;
      this.$refs.bg.height = this.$refs.mark.height = this.options.canvasHeight;
      this._fetchImageAndDraw(imgSrc);
    },
    _fetchImageAndDraw(src) {
      let { width, canvasHeight, r, markWidth, markHeight } = this.options;
      const img = new Image();
      img.crossOrigin = "Anonymous";
      img.src = src;
      img.onload = () => {
        console.log("onload");
        let { x, y } = this.getMarkPostion();
        this.targetX = x;
        this.draw(this.bg_ctx, x, y, "fill");
        this.draw(this.mark_ctx, x, y, "clip");
        this.bg_ctx.drawImage(img, 0, 0, width, canvasHeight);
        this.mark_ctx.drawImage(img, 0, 0, width, canvasHeight);
        const imageData = this.mark_ctx.getImageData(
          x,
          y - r * 2 - 2,
          markWidth,
          markHeight
        );
        this.$refs.mark.width = markWidth;
        this.mark_ctx.putImageData(imageData, 0, y - r * 2 - 2);
      };
    },
    draw(ctx, x, y, operation) {
      let PI = Math.PI;
      let { r, markWidth, markHeight } = this.options;
      var wl = markWidth - r * 2 - 2;
      var hl = markHeight - r * 2 - 2;
      ctx.beginPath();
      ctx.moveTo(x, y);
      ctx.arc(x + wl / 2, y - r + 2, r, 0.72 * PI, 2.26 * PI);
      ctx.lineTo(x + wl, y);
      ctx.arc(x + wl + r - 2, y + hl / 2, r, 1.21 * PI, 2.78 * PI);
      ctx.lineTo(x + wl, y + hl);
      ctx.lineTo(x, y + hl);
      ctx.arc(x + r - 2, y + hl / 2, r + 0.4, 2.76 * PI, 1.24 * PI, true);
      ctx.lineTo(x, y);
      ctx.lineWidth = 2;
      ctx.fillStyle = "rgba(255, 255, 255, 0.7)";
      ctx.strokeStyle = "rgba(255, 255, 255, 0.7)";
      ctx.stroke();
      ctx[operation]();
      ctx.globalCompositeOperation = "destination-over";
    },
    vertify() {
      let that = this;
      if (Math.abs(this.$options.slideLeft - this.targetX) < 3) {
        this.options.vertifyCallback &&
          this.options.vertifyCallback().then(
            isSuccess => {
              this.isFail = false;
              this.isSuccess = true;
            },
            reject => {
              this.isSuccess = false;
              this.isFail = true;
              setTimeout(() => {
                that.$refs.slide.style.left = that.$refs.mark.style.left = that.$refs.slided.style.width =
                  0 + "px";
                that.isFail = false;
              }, 500);
            }
          );
      } else {
        this.options.vertifyErrorCallback();
        this.isSuccess = false;
        this.isFail = true;
        setTimeout(() => {
          that.$refs.slide.style.left = that.$refs.mark.style.left = that.$refs.slided.style.width =
            0 + "px";
          that.isFail = false;
        }, 500);
      }
    }
  },
  created() {},
  mounted() {
    document.addEventListener("mousemove", this.mousemove);
    document.addEventListener("mouseup", this.mouseup);
    this.$refs.bg.width = this.$refs.mark.width = this.options.width;
    this.$refs.bg.height = this.$refs.mark.height = this.options.canvasHeight;
    this.bg_ctx = this.$refs.bg.getContext("2d");
    this.mark_ctx = this.$refs.mark.getContext("2d");
    this._fetchImageAndDraw(this.options.src);
  }
};
</script>

<style scope>
#ptVertify {
  position: relative;
}

#ptVertify .refresh:hover {
  color: rgb(109, 217, 221);
}
#ptVertify .refresh {
  position: absolute;
  right: 0;
  color: white;
  text-shadow: 0px 0px 3px #ccc;
  cursor: pointer;
  user-select: none;
}

#vertify_mark {
  position: absolute;
  left: 0;
  top: 0;
}

.slider-wrap {
  -webkit-user-select: none;
  user-select: none;
  position: relative;
}
.slider {
  display: block;
  text-align: center;
  color: transparent;
  background: linear-gradient(to right, black 40%, white 50%, black 60%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  text-size-adjust: none;
  border: 1px solid rgb(143, 143, 143);
  padding: 10px;
  font-size: 18px;
  /* transition: background-position .5s; */
  animation: slide linear infinite;
  animation-duration: 1.5s;
  /* position: relative; */
}

.slide {
  -webkit-user-select: none;
  cursor: pointer;
  color: rgb(138, 138, 138);
  font-size: 16px;
  position: absolute;
  left: var(--slideX);
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #eee;
  width: 50px;
  height: 100%;
  /* transition: left 0.1s; */
  /* border: 1px solid green; */
  box-shadow: 0px 0px 1px 1px rgb(116, 116, 116);
  z-index: 5;
  transition: color 0.5s, box-shadow 0.5s;
}

.slided {
  position: absolute;
  top: 0;
  left: 0;
  width: 0;
  height: 100%;
  background: rgba(129, 129, 253, 0.5);
  /* background: rgb(255, 113, 113); */
}

.slide:hover {
  color: rgb(129, 129, 253);
  box-shadow: 0px 0px 1px 1px rgb(129, 129, 253);
}

.slided.fail {
  background: rgba(255, 128, 128, 0.5);
}
.slided.success {
  background: rgba(192, 255, 163, 0.5);
}

.slide.fail {
  color: rgb(255, 128, 128);
  box-shadow: 0px 0px 1px 1px rgb(255, 128, 128);
}
.slide.success {
  color: rgb(94, 212, 39);
  box-shadow: 0px 0px 1px 1px rgb(94, 212, 39);
}

@keyframes slide {
  0% {
    background-position: -200px 0;
  }

  100% {
    background-position: 200px 0;
  }
}
</style>
