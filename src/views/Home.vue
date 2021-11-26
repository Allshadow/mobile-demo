<template>
  <div class="home">
    <van-button type="danger" @click="toSign">电子签名</van-button>
    <base-signature
      title="学生电子签字"
      :isShow.sync="signShow"
      @done="done">
    </base-signature>
  </div>
</template>

<script>
import BaseSignature from '@/components/BaseSignature.vue'

export default {
  name: 'Home',
  components: {
    BaseSignature
  },
  data () {
    return {
      signShow: false
    }
  },

  methods: {
    done (val) {
      // 签字返回的内容
    },

    toSign () {
      this.signShow = true
    }
  }
}
</script>
