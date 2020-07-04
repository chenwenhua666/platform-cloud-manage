<template>
  <div class="filter-container">
    <el-form ref="form" :model="form" :rules="rules" label-position="right" label-width="80px" class="form">
      <el-form-item :label="$t('table.mail.validation')" prop="email">
        <el-input v-model="form.email" class="filter-item search-item" />
        <el-button class="filter-item" type="primary" :loading="codeLoading" :disabled="isDisabled" size="small" @click="sendCode">{{ buttonName }}</el-button>
      </el-form-item>
      <el-form-item :label="$t('login.code')" prop="code">
        <el-input v-model="form.code" class="filter-item search-item" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" plain :loading="buttonLoading" @click="download">{{ $t('table.report.download') }}</el-button>
        <el-button type="success" plain @click="view">{{ $t('table.report.view') }}</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
import { validEmail } from '@/utils/my-validate'

export default {
  props: {
    user: {
      type: Object,
      default: () => {
        return {
          name: '',
          email: ''
        }
      }
    }
  },
  data() {
    return {
      form: this.initForm(),
      buttonName: this.$t('table.mail.getCode'),
      buttonLoading: false,
      codeLoading: false,
      isDisabled: false,
      time: 60,
      rules: {
        email: [
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { validator: (rule, value, callback) => {
            if (value !== '' && !validEmail(value)) {
              callback(this.$t('rules.email'))
            } else {
              callback()
            }
          }, trigger: 'blur' }],
        code: { required: true, message: this.$t('rules.require'), trigger: 'blur' }
      }
    }
  },
  methods: {
    sendCode() {
      if (this.form.email && validEmail(this.form.email)) {
        this.codeLoading = true
        this.buttonName = this.$t('table.mail.sending')
        const _this = this
        this.$get('statistics/mail/template', {
          address: this.form.email
        }).then(() => {
          this.$message({
            showClose: true,
            message: this.$t('table.mail.validity'),
            type: 'success'
          })
          this.codeLoading = false
          this.isDisabled = true
          const laterMessage = this.$t('table.mail.later')
          const resendMessage = this.$t('table.mail.resend')
          this.buttonName = this.time-- + laterMessage
          this.timer = window.setInterval(() => {
            _this.buttonName = _this.time + laterMessage
            --_this.time
            if (_this.time < 0) {
              _this.buttonName = resendMessage
              _this.time = 60
              _this.isDisabled = false
              window.clearInterval(_this.timer)
            }
          }, 1000)
        }).catch(() => {
          this.resetForm()
        })
      }
    },
    resetForm() {
      this.$refs.form.clearValidate()
      this.$refs.form.resetFields()
      window.clearInterval(this.timer)
      this.time = 60
      this.buttonName = this.$t('table.mail.getCode')
      this.isDisabled = false
      this.codeLoading = false
      this.form = this.initForm()
    },
    download() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          this.buttonLoading = true
          this.$download('statistics/report/download', {
            ...this.form
          }, `${new Date().getTime()}_report.pdf`).then(() => {
            this.buttonLoading = false
            this.resetForm()
          }).catch(() => {
            this.buttonLoading = false
          })
        } else {
          return false
        }
      })
    },
    view() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          const routeData = this.$router.resolve({ path: '/pdf/view', query: this.form })
          window.open(routeData.href, '_blank')
        } else {
          return false
        }
      })
    },
    initForm() {
      return {
        email: this.user.email,
        code: ''
      }
    }
  }
}
</script>
