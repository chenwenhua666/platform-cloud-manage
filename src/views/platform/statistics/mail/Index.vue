<template>
  <div class="app-container">
    <div class="filter-container">
      <el-form ref="form" :model="form" :rules="rules" label-width="100px">
        <el-form-item :label="$t('table.mail.title')" prop="title">
          <el-input v-model="form.title" />
        </el-form-item>
        <el-form-item
          v-for="(item, index) in mailAddress"
          :key="item.key"
          :label="$t('table.mail.receiver') + (index === 0 ? '': index)"
        >
          <el-input v-model="item.value" class="filter-item search-item" />
          <el-button icon="el-icon-plus" class="filter-item" @click="add" />
          <el-button icon="el-icon-minus" class="filter-item" @click.prevent="remove(item)" />
        </el-form-item>
        <el-form-item :label="$t('table.mail.content')" prop="content">
          <div>
            <tinymce ref="tinymce" v-model="form.content" :height="300" />
          </div>
        </el-form-item>
        <el-form-item v-has-permission="['mail:send']">
          <el-button :loading="loading" size="medium" type="primary" plain @click="doSubmit">{{ $t('table.mail.send') }}</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import Tinymce from '@/components/Tinymce'
import { randomNum } from '@/utils'
import { validEmail } from '@/utils/my-validate'

export default {
  name: 'MailManage',
  components: { Tinymce },
  data() {
    return {
      loading: false,
      form: this.initForm(),
      mailAddress: this.initMailAddress(),
      rules: {
        title: { required: true, message: this.$t('rules.require'), trigger: 'blur' },
        content: { required: true, message: this.$t('rules.require'), trigger: 'blur' }
      }
    }
  },
  methods: {
    initForm() {
      return {
        title: '',
        receiver: [],
        content: ''
      }
    },
    initMailAddress() {
      return [{
        value: '',
        key: randomNum(24, 16)
      }]
    },
    add() {
      this.mailAddress.push({
        value: '',
        key: randomNum(24, 16)
      })
    },
    remove(item) {
      const index = this.mailAddress.indexOf(item)
      if (index !== -1 && this.mailAddress.length !== 1) {
        this.mailAddress.splice(index, 1)
      } else {
        this.$message({
          message: this.$t('table.mail.atLeast'),
          type: 'warning'
        })
      }
    },
    doSubmit() {
      this.$refs.form.validate((valid) => {
        this.form.receiver = []
        if (valid) {
          let sub = false
          this.mailAddress.forEach((data, index) => {
            if (data.value === '') {
              this.$message({
                message: this.$t('table.mail.require'),
                type: 'warning'
              })
              sub = true
            } else if (validEmail(data.value)) {
              this.form.receiver.push(data.value)
            } else {
              this.$message({
                message: this.$t('rules.email'),
                type: 'warning'
              })
              sub = true
            }
          })
          if (sub) { return false }
          this.loading = true
          this.$post('statistics/mail/send', { ...this.form }).then(() => {
            this.$notify({
              title: this.$t('table.mail.success'),
              type: 'success',
              duration: 2500
            })
            this.loading = false
            this.reset()
          }).catch(() => {
            this.loading = false
          })
        } else {
          return false
        }
      })
    },
    reset() {
      this.$refs.form.clearValidate()
      this.$refs.form.resetFields()
      this.form = this.initForm()
      this.mailAddress = this.initMailAddress()
      this.$refs.tinymce.setContent('')
    }
  }
}
</script>
<style lang="scss" scoped>
  .editor-content{
    margin-top: 20px;
  }
</style>
