<template>
  <div class="app-container">
    <div class="filter-container">
      <div class="button-group">
        <div class="filter-item"><span class="requireText">*</span>{{ $t('table.sql.datasource') }}</div>
        <el-select v-model="queryParams.datasource" class="filter-item search-item">
          <el-option
            v-for="item in dataSourcesName"
            :key="item.databaseName"
            :label="item.databaseName"
            :value="item.dataSourceName"
          />
        </el-select>
        <el-button v-hasPermission="['sql:run']" class="filter-item" type="primary" @click="run">{{ $t('table.sql.run') }}</el-button>
        <el-button class="filter-item" type="danger" @click="reset">{{ $t('table.reset') }}</el-button>
        <el-button class="filter-item" type="success" @click="formaterSql(basicInfoForm.sqlMain)">{{ $t('table.sql.format') }}</el-button>
      </div>
      <SqlEditor
        ref="sqleditor"
        :value="basicInfoForm.sqlMain"
        @changeTextarea="changeTextarea($event)"
      />
      <el-divider><i class="el-icon-data-analysis" /></el-divider>

      <el-table
        ref="table"
        :key="tableKey"
        v-loading="loading"
        :data="list"
        max-height="400"
        border
        fit
        style="width: 100%;"
      >
        <template v-for="(value, key) in children">
          <el-table-column :key="key" :label="key" :prop="key" align="center" :show-overflow-tooltip="true" />
        </template>
      </el-table>
    </div>
  </div>
</template>
<script>
import sqlFormatter from 'sql-formatter'
import SqlEditor from '@/components/SqlEditor'
export default {
  components: {
    SqlEditor
  },
  data() {
    return {
      basicInfoForm: {
        sqlMain: ''
      },
      queryParams: {},
      dataSourcesName: [],
      list: null,
      children: null,
      tableKey: 0,
      loading: false
    }
  },
  mounted() {
    this.getDataSources()
  },
  methods: {
    changeTextarea(val) {
      this.$set(this.basicInfoForm, 'sqlMain', val)
    },
    formaterSql(val) {
      const dom = this.$refs.sqleditor
      dom.editor.setValue(sqlFormatter.format(dom.editor.getValue()))
    },
    getDataSources() {
      this.$get('statistics/sql/datasource').then((r) => {
        this.dataSourcesName = r.data.data
      }).catch(() => {
        this.$message({
          message: this.$t('tips.getDataFail'),
          type: 'error'
        })
      })
    },
    run() {
      if (!this.queryParams.datasource) {
        this.$message({
          message: this.$t('tips.noDataSelected'),
          type: 'error'
        })
        return
      }

      if (!this.basicInfoForm.sqlMain) {
        this.$message({
          message: this.$t('rules.require'),
          type: 'error'
        })
        return
      }

      this.loading = true
      this.children = null
      this.list = null
      this.$post('statistics/sql/run', {
        ...this.basicInfoForm,
        ...this.queryParams
      }).then((r) => {
        const data = r.data.data
        this.list = data
        if (data) {
          this.children = data[0]
        }
        this.loading = false
        if (r.data.message) {
          this.$message({
            message: this.$t('tips.executeSuccess'),
            type: 'success'
          })
        }
      }).catch(() => {
        this.loading = false
      })
    },
    reset() {
      this.basicInfoForm.sqlMain = ''
      this.$refs.sqleditor.editor.setValue('')
      this.children = null
      this.list = null
    }
  }
}
</script>
<style lang="scss" scoped>
  .button-group {
    text-align: center;
  }
  .requireText {
    color: #F56C6C;
    margin-right: 4px
  }
</style>
