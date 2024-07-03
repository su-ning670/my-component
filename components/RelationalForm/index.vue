<template>
  <div class="relational-form">
    <el-form ref="relationForm" :model="formData" size="mini">
      <relation
        ref="relation"
        role="relation"
        :value="formData"
        :settings="settings"
        :rules="settingsConfig.rules"
        :item-default-value="settingsConfig.defaultItemValue"
        v-on="$listeners"
      >
        <template
          v-for="sl in settingsConfig.slotKeys"
          :slot="sl"
          slot-scope="{ scope }"
        >
          <slot :name="sl" :scope="scope" />
        </template>
      </relation>
    </el-form>

    <el-button type="primary" class="add-rule-btn" @click="handleAdd">
      添加规则
    </el-button>
  </div>
</template>

<script>
import Relation from './components/item'

export default {
  name: 'RelationForm',
  components: {
    Relation
  },
  props: {
    // filter 里面的数据
    value: {
      type: Object,
      default: () => ({}),
      required: true
    },
    settings: {
      type: Array,
      default: () => [],
      required: true
    },
    getTypeValue: {
      type: Function,
      default: null
    }
  },
  data() {
    return {
      formData: {}
    }
  },
  computed: {
    settingsConfig() {
      const slot = []
      const rules = {}
      const defaultItemValue = {}
      this.settings.forEach(item => {
        if (item.slot === true) {
          slot.push(item.name)
          rules[item.name] = item.rules.reduce((pre, curr) => {
            pre.push({ trigger: 'blur', ...curr })
            return pre
          }, [])
          defaultItemValue[item.name] = this.getTypeValueOrigin(item.dataType)
        }
      })
      return {
        slotKeys: slot,
        rules,
        defaultItemValue
      }
    }
  },
  mounted() {
    this.$emit('getFormValue', () => this.$refs['relation'].value) // 没有校验的表单数据
    if (!this.value) return
    this.formData = { ...this.formatData(this.value) } // 初始化拿到的数据格式
  },
  methods: {
    /**
     * 新增规则
     */
    handleAdd() {
      if (!this.formData.conditions) this.formData.conditions = []
      const lastOne = this.formData.conditions[
        this.formData.conditions.length - 1
      ]

      if (!this.formData.conditions.length) {
        // 没有子级
        // eslint-disable-next-line no-unused-vars
        const { relation, conditions, ...rest } = this.formData
        this.formData = {
          relation: relation || 'and',
          conditions: [
            {
              ...this.settingsConfig.defaultItemValue,
              ...rest
            },
            {
              ...this.settingsConfig.defaultItemValue,
              sortIndex: this.formData.sortIndex + 1
            }
          ]
        }
      } else {
        // 有多条
        this.formData.conditions.push({
          ...this.settingsConfig.defaultItemValue,
          sortIndex: lastOne.sortIndex + 1
        })
      }

      this.$emit('onChange', { type: 'addOne', value: this.formData })
    },
    /**
     * 格式化数据，用于正常显示从后台拿到的数据
     * @param {*} value 业务组件传过来的数据
     */
    formatData(value) {
      if (Object.keys(value).length === 0) {
        // 新增
        return { sortIndex: 0 }
      } else {
        // 编辑
        const result = value.conditions.sort(
          (a, b) => a.sortIndex - b.sortIndex
        )
        return { relation: 'and', ...value, conditions: result, sortIndex: 0 }
      }
    },
    /**
     * 表单提交
     */
    submitForm() {
      return new Promise((resolve, reject) => {
        this.$refs.relationForm.validate(valid => {
          if (valid) {
            resolve(this.$refs['relation'].value) // 校验通过把表单值传给父组件
          } else {
            setTimeout(() => {
              reject('error')
            })
          }
        })
      })
    },
    /**
     * 根据表单的数据类型动态获取默认的一行对象
     * @param {*} type 字段类型
     */
    getTypeValueOrigin(type) {
      if (this.getTypeValue) return this.getTypeValue(type)
      let val = ''
      switch (type) {
        case 'array':
          val = []
          break
        case 'object':
          val = {}
          break
        case 'boolean':
          val = false
          break
        case 'number':
          val = 0
          break
        default:
      }
      return val
    }
  }
}
</script>

<style lang="scss">
.relational-form .el-form .el-form-item__content {
  .el-select,
  .el-input,
  .el-textarea,
  .el-tree,
  .vue-treeselect,
  .tinymce-container {
    width: 97%;
  }
}
.add-rule-btn {
  width: 100%;
  color: $primary !important;
  background: #fff !important;
  border: 1px dashed $primary;
}
</style>
