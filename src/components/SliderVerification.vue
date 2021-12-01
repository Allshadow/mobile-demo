<template>
  <!-- 滑块验证功能 -->
  <div class="sliderModel" v-if="visible">
    <div class="cont">
      <div class="title">滑块验证</div>
      <div class="slider-refresh">
        <span @click="handleRefresh">刷新</span>
        <span @click="handleClose">关闭</span>
      </div>
      <!-- canvas 图片 -->
      <div class="imgWrap">
        <canvas ref="sliderBlock" class="slider-block"></canvas>
        <canvas ref="codeImg" class="code-img"></canvas>
      </div>
      <!-- 滑块 -->
      <div class="sliderBox">
        <div class="sliderF">
          <div class="sliderS" @touchstart.prevent="handleTouch">
            <div class="btn">&gt;&gt;</div>
          </div>
        </div>
        <div class="bgC">
          {{ tips }}
          <div class="bgC_left"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    isShow: {
      type: Boolean,
      default: false
    },
    options: {
      // 传入的参数不影响组件
      type: Object,
      default: () => ({})
    },
    imgList: { // 背景图片
      type: Array,
      default: () => {
        const img1 = require('../assets/slider-verification/1.jpg')
        const img2 = require('../assets/slider-verification/2.jpg')
        const img3 = require('../assets/slider-verification/3.jpg')
        const img4 = require('../assets/slider-verification/4.jpg')
        const img5 = require('../assets/slider-verification/5.jpg')
        const img6 = require('../assets/slider-verification/6.jpg')

        return [img1, img2, img3, img4, img5, img6]
      }
    }
  },
  data () {
    return {
      // 滑块x轴数据
      slider: {
        mx: 0,
        bx: 0
      },
      tips: '',
      visible: false,
      mainDom: '',
      blockDom: ''
    }
  },
  watch: {
    isShow: {
      handler (newVal) {
        this.visible = newVal
        if (newVal === true) {
          this.tips = '拖动左边滑块完成上方拼图'
          this.$nextTick(() => {
            this.getDom()
            this.canvasInit()
          })
        }
      },
      immediate: true
    }
  },

  methods: {
    // 获取 dom
    getDom () {
      this.mainDom = this.$refs.codeImg
      this.blockDom = this.$refs.sliderBlock
    },

    handleClose () {
      this.$emit('update:isShow', false)
    },

    // 刷新
    handleRefresh () {
      this.canvasInit()
    },

    // 移动端事件
    handleTouch (e) {
      const ev = e || window.event
      const dom = ev.target // dom元素

      const downCoordinate = {
        x: ev.touches[0].pageX,
        y: ev.touches[0].pageY
      }
      // 正确的滑块数据
      const checkx = Number(this.slider.mx) - 0
      // x轴数据
      let x = 0
      const move = (moveEV) => {
        x = moveEV.touches[0].pageX - downCoordinate.x
        // //y = moveEV.y - downCoordinate.y;
        if (x >= 251 || x <= 0) return false
        dom.style.left = x + 'px'
        // dom.style.top = y + "px";
        this.blockDom.style.left = x + 'px'
      }

      const up = () => {
        document.removeEventListener('touchmove', move)
        document.removeEventListener('touchend', up)
        dom.style.left = ''

        console.log(x, checkx)
        const max = checkx - 5
        const min = checkx - 15
        // 允许正负误差1
        if ((max >= x && x >= min) || x === checkx) {
          this.tips = '验证成功'
          this.$emit('done', this.options.type)
        } else {
          this.tips = '验证失败，请重试'
          this.blockDom.style.left = 0
          this.canvasInit()
        }
      }

      document.addEventListener('touchmove', move)
      document.addEventListener('touchend', up)
    },

    // 拼图验证码初始化
    canvasInit () {
      // 生成指定区间的随机数
      const random = (min, max) => {
        return Math.floor(Math.random() * (max - min + 1) + min)
      }
      // x: 254, y: 109
      const mx = random(127, 230)
      const bx = random(10, 128)
      const y = random(10, 99)

      this.slider = { mx, bx }

      this.draw(mx, bx, y)
    },

    draw (mx = 200, bx = 20, y = 50) {
      const bg = this.mainDom.getContext('2d')
      const block = this.blockDom.getContext('2d')

      const width = this.mainDom.width
      const height = this.mainDom.height

      // 重新赋值，让canvas进行重新绘制
      this.blockDom.height = height
      this.mainDom.height = height
      this.mainDom.width = width
      // 随机背景图片
      const randomImg = () => {
        const num = Math.floor(Math.random() * this.imgList.length)
        return this.imgList[num]
      }

      const imgsrc = randomImg()
      const img = document.createElement('img')
      img.style.objectFit = 'scale-down'
      img.src = imgsrc
      img.onload = () => {
        bg.drawImage(img, 0, 0, width, height)
        block.drawImage(img, 0, 0, width, height)
        const ImageData = block.getImageData(mx, y, width, height)
        block.putImageData(ImageData, 0, y)
      }

      const mainxy = { x: mx, y: y, r: 9 }
      const blockxy = { x: mx, y: y, r: 9 }
      this.drawBlock(bg, mainxy, 'fill')
      this.drawBlock(block, blockxy, 'clip')
    },

    // 绘制拼图
    drawBlock (ctx, xy = { x: 254, y: 109, r: 9 }, type) {
      const x = xy.x
      const y = xy.y
      const r = xy.r
      const w = 40
      const PI = Math.PI
      // 绘制
      ctx.beginPath()
      // left
      ctx.moveTo(x, y)
      // top
      ctx.arc(x + (w + 5) / 2, y, r, -PI, 0, true)
      ctx.lineTo(x + w + 5, y)
      // right
      ctx.arc(x + w + 5, y + w / 2, r, 1.5 * PI, 0.5 * PI, false)
      ctx.lineTo(x + w + 5, y + w)
      // bottom
      ctx.arc(x + (w + 5) / 2, y + w, r, 0, PI, false)
      ctx.lineTo(x, y + w)
      ctx.arc(x, y + w / 2, r, 0.5 * PI, 1.5 * PI, true)
      ctx.lineTo(x, y)
      // 修饰，没有会看不出效果
      ctx.lineWidth = 1
      ctx.fillStyle = 'rgba(255, 255, 255, 0.5)'
      ctx.strokeStyle = 'rgba(255, 255, 255, 0.5)'
      ctx.stroke()
      ctx[type]()
      ctx.globalCompositeOperation = 'xor'
    }
  }
}
</script>

<style scoped lang="scss">
.sliderModel {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.title {
  width: 100%;
  height: 60px;
  font-size: 18px;
  color: #333;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cont {
  position: relative;
  background: #fff;
  width: 300px;
  border-radius: 8px;
  overflow: hidden;
  padding-bottom: 10px;
}

.imgWrap {
  position: relative;
  width: 280px;
  height: 150px;
  margin: 0 auto;
  overflow: hidden;
  .code-img,
  .slider-block {
    border-radius: 8px;
    height: inherit;
  }

  .code-img {
    width: 280px;
    margin: 0 auto;
  }

  .slider-block {
    position: absolute;
    z-index: 4000;
    left: 0;
  }
}

.slider-refresh {
  font-size: 14px;
  position: absolute;
  top: 20px;
  right: 20px;
  cursor: pointer;
  color: green;
  span{
    padding: 0 2px;
  }
}

.img {
  display: block;
  width: 100%;
  height: 100%;
}

.sliderOver {
  position: absolute;
  left: 0;
  top: 0;
  width: 50px;
  height: 50px;
  background: #ddd;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.3);
}

.smartImg {
  position: absolute;
  z-index: 2;
  left: 0;
  top: 0;
  width: 50px;
  height: 50px;
  overflow: hidden;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
}

.simg {
  position: absolute;
  display: block;
  width: 280px;
  height: 150px;
}

.sliderBox {
  width: 280px;
  margin: 15px auto 0;
  height: 36px;
  position: relative;
}

.sliderF {
  width: 100%;
  height: 100%;
  z-index: 3;
}

.sliderS {
  cursor: pointer;
  position: absolute;
  left: 0;
  top: 0;
  z-index: 2;
  height: 36px;
  width: 36px;
  border-radius: 36px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.icon {
  width: 20px;
  height: 20px;
}

.bgC {
  position: absolute;
  z-index: 1;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 100%;
  height: 30px;
  border-radius: 30px;
  line-height: 30px;
  font-size: 14px;
  color: #999999;
  box-shadow: inset 0 0 4px #ccc;
  text-align: center;
  overflow: hidden;
}

.bgC_left {
  position: absolute;
  left: 0px;
  top: 50%;
  transform: translateY(-50%);
  width: 0;
  height: 28px;
  border-top-left-radius: 28px;
  border-bottom-left-radius: 28px;
  line-height: 28px;
  font-size: 14px;
  background-color: #eee;
  box-shadow: inset 0 0 4px #ccc;
  text-align: center;
}

.showMessage {
  text-align: center;
  font-size: 14px;
  height: 30px;
  line-height: 30px;
}

#closeBtn {
  position: fixed;
  z-index: 10;
  bottom: 10px;
  left: 50%;
}

.btn {
  width: 36px;
  height: 36px;
  position: absolute;
  border: 1px solid #ccc;
  cursor: move;
  font-family: "宋体";
  text-align: center;
  line-height: 36px;
  background-color: #fff;
  user-select: none;
  color: #666;
  font-size: 16px;
}
</style>
