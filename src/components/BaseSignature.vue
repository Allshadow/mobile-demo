<template>
  <!-- 移动端电子签名 -->
  <div class="base-signature">
    <van-dialog
      v-model="signShow"
      show-cancel-button
      @cancel="handleCancel"
      @confirm="handleConfirm"
      :beforeClose="beforeClose"
    >
      <div slot="title" class="sign-header">
        <span>{{ title }}</span>
        <span style="color: #1989fa" @click="handleClear">
          <van-icon name="revoke" />
        </span>
      </div>
      <div>
        <!-- 注意这个宽高要设置，不然会出问题 -->
        <vueSignature
          ref="signature"
          :sigOption="option"
          :w="'316px'"
          :h="'400px'"
          :disabled="disabled"
          class="signature"
        >
        </vueSignature>
      </div>
    </van-dialog>
  </div>
</template>

<script>
import vueSignature from 'vue-signature'

export default {
  name: 'BaseSignature',
  props: {
    title: {
      type: String,
      default: '电子签字'
    },

    isShow: {
      type: Boolean,
      default: false
    },

    option: {
      type: Object,
      default: () => ({
        penColor: 'rgb(0, 0, 0)',
        backgroundColor: 'rgb(255,255,255)'
      })
    },

    disabled: {
      type: Boolean,
      default: false
    }
  },
  components: {
    vueSignature
  },
  data () {
    return {}
  },
  computed: {
    signShow: {
      get () {
        return this.isShow
      },

      set () {}
    }
  },
  watch: {},
  methods: {
    beforeClose (action, done) {
      if (action === 'confirm') {
        return done(false)
      } else {
        return done()
      }
    },

    handleCancel () {
      this.handleClear()
      this.$emit('update:isShow', false)
    },

    handleConfirm () {
      const isEmpty = this.$refs.signature.isEmpty()
      if (isEmpty) {
        this.$toast({ message: '请签字', position: 'bottom' })
      } else {
        const electronicSign = this.$refs.signature.save()
        this.handleCancel()
        this.$emit('done', electronicSign)
      }
    },

    // 清除签名
    handleClear () {
      this.$refs.signature.clear()
    }
  },
  created () {},
  mounted () {}
}
</script>
<style scoped lang="scss">
.sign-header {
  display: flex;
  justify-content: space-between;
  text-align: left;
  padding: 0 20px;
  margin-bottom: 15px;
}

.signature {
  border: 1px solid #1989fa !important;
  margin: 0 auto;
  border-radius: 6px;
}
</style>
