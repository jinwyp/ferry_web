<template>
  <div>
    <el-form
      ref="generateForm"
      label-suffix=":"
      :size="data.config.size"
      :model="models"
      :rules="rules"
      :label-position="data.config.labelPosition"
      :label-width="data.config.labelWidth + 'px'"
    >
      <template v-for="item in data.list">

        <template v-if="item.type == 'grid'">
          <el-row
            :key="item.key"
            type="flex"
            :gutter="item.options.gutter ? item.options.gutter : 0"
            :justify="item.options.justify"
            :align="item.options.align"
          >
            <el-col v-for="(col, colIndex) in item.columns" :key="colIndex" :span="col.span">

              <template v-for="citem in col.list">
                <el-form-item v-if="citem.type=='blank'" :key="citem.key" :label="citem.name" :prop="citem.model">
                  <slot :name="citem.model" :model="models" />
                </el-form-item>
                <genetate-form-item
                  v-else
                  :key="citem.key"
                  :preview="preview"
                  :models.sync="models"
                  :remote="remote"
                  :rules="rules"
                  :widget="citem"
                  :data="data"
                  @input-change="onInputChange"
                />
              </template>
            </el-col>
          </el-row>
        </template>

        <template v-else-if="item.type == 'blank'">
          <el-form-item :key="item.key" :label="item.name" :prop="item.model">
            <slot :name="item.model" :model="models" />
          </el-form-item>
        </template>
        <!-- 子表单 -->
        <template v-if="item.type === 'subform'">
          <el-form-item
            :key="item.key"
            :label-width="!item.options.labelWidthStatus?'0px': item.options.labelWidth + 'px'"
            :label="!item.options.labelWidthStatus?'':item.name"
            :prop="item.model"
          >
            <el-table
              :data="tableData"
              border
              style="width: 100%"
            >
              <el-table-column
                width="50"
              >
                <template slot="header">
                  <i style="font-size: 25px; color: #409EFF;cursor:pointer;" class="el-icon-circle-plus" @click="addCol(item)" />
                </template>
                <template>
                  <i style="font-size: 25px; color: red" class="el-icon-remove" />
                </template>

              </el-table-column>
              <el-table-column
                v-for="v in item.columns.list"
                :key="v.key"
                :prop="v.key"
                :label="v.name"
                width="250"
              >
                <template>
                  <genetate-form-item
                    :preview="preview"
                    :models.sync="models"
                    :rules="rules"
                    :widget="v"
                    :remote="remote"
                    :data="data"
                    :disabled="disabled"
                    :is-label="false"
                    @input-change="onSubformInputChange"
                  />
                </template>
              </el-table-column>
            </el-table>
          </el-form-item>
        </template>

        <template v-else>
          <genetate-form-item
            :key="item.key"
            :preview="preview"
            :models.sync="models"
            :rules="rules"
            :widget="item"
            :remote="remote"
            :data="data"
            :disabled="disabled"
            @input-change="onInputChange"
          />
        </template>

      </template>
    </el-form>
  </div>
</template>

<script>
import GenetateFormItem from './GenerateFormItem'

export default {
  name: 'FmGenerateForm',
  components: {
    GenetateFormItem
  },
  /* eslint-disable */
  props: ['data', 'remote', 'value', 'insite', 'disabled', 'preview'],
  data() {
    return {
      tableData: [],
      models: {},
      rules: {}
    }
  },
  watch: {
    data: {
      deep: true,
      handler(val) {
        this.generateModle(val.list)
      }
    },
    value: {
      deep: true,
      handler(val) {
        this.models = { ...this.models, ...val }
      }
    }
  },
  created() {
    this.generateModle(this.data.list)
  },
  mounted() {
  },
  methods: {
    addCol(item) {
      var j = {}
      j["id"] = this.tableData.length + 1
      this.tableData.push(j)
    },
    generateModle(genList) {
      for (let i = 0; i < genList.length; i++) {
        if (genList[i].type === 'grid') {
          genList[i].columns.forEach(item => {
            this.generateModle(item.list)
          })
        } else if (genList[i].type === 'subform') {
          // this.generateModle(genList[i].columns.list)
        } else {
          if (this.value && Object.keys(this.value).indexOf(genList[i].model) >= 0) {
            this.models[genList[i].model] = this.value[genList[i].model]
          } else {
            if (genList[i].type === 'blank') {
              this.$set(this.models, genList[i].model, genList[i].options.defaultType === 'String' ? '' : (genList[i].options.defaultType === 'Object' ? {} : []))
            } else {
              this.models[genList[i].model] = genList[i].options.defaultValue
            }
          }

          if (this.rules[genList[i].model]) {
            this.rules[genList[i].model] = [...this.rules[genList[i].model], ...genList[i].rules.map(item => {
              if (item.pattern) {
                return { ...item, pattern: eval(item.pattern) }
              } else {
                return { ...item }
              }
            })]
          } else {
            this.rules[genList[i].model] = [...genList[i].rules.map(item => {
              if (item.pattern) {
                return { ...item, pattern: eval(item.pattern) }
              } else {
                return { ...item }
              }
            })]
          }
        }
      }
    },
    getData() {
      return new Promise((resolve, reject) => {
        this.$refs.generateForm.validate(valid => {
          if (valid) {
            resolve(this.models)
          } else {
            reject(new Error(this.$t('fm.message.validError')).message)
          }
        })
      })
    },
    reset() {
      this.$refs.generateForm.resetFields()
    },
    onSubformInputChange(value, field) {
      this.$emit('on-change', field, value, this.models)
    },
    onInputChange(value, field) {
      this.$emit('on-change', field, value, this.models)
    },
    refresh() {

    }
  }
}
</script>

<style lang="scss">
// @import '../styles/cover.scss';
</style>
