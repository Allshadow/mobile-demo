<template>
  <div class="home">
    <van-button type="danger" @click="toSign">电子签名</van-button>
    <van-button type="danger" @click="toConfirm">滑块验证</van-button>
    <!-- 电子签名开始 -->
    <base-signature title="学生电子签字" :isShow.sync="signShow" @done="done">
    </base-signature>
    <!-- 电子签名结束 -->
    <!-- 滑块验证开始 -->
    <slider-verification
      :isShow.sync="sliderShow"
      @done="handleConfirm"
    >
    </slider-verification>
    <!-- 滑块验证结束 -->
  </div>
</template>

<script>
import BaseSignature from '@/components/BaseSignature.vue'
import SliderVerification from '@/components/SliderVerification.vue'

export default {
  name: 'Home',
  components: {
    BaseSignature,
    SliderVerification
  },
  data () {
    return {
      signShow: false,
      sliderShow: false
    }
  },

  methods: {
    done (val) {
      // 签字返回的内容
    },

    toSign () {
      this.signShow = true
    },

    // 滑块验证成功后回调
    handleConfirm (val) {
      this.sliderShow = false
    },

    toConfirm () {
      this.sliderShow = true
    }
  }
}
</script>
