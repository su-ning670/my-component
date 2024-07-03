<template>
  <div class="relation-item-node" style="position: relative;">
    <!-- 渲染一行 -->
    <template v-if="!conditionsLen">
      <!-- 一行组件 -->
      <el-row class="relation-item-node-block">
        <el-col class="relation-item-node-base">
          <template v-for="(item, itemIndex) in settings">
            <!-- 基础节点 -->
            <el-col
              :key="dataKey + '_' + item.name"
              :span="item.span"
              :class="item.name + itemIndex"
            >
              <el-form-item
                :key="dataKey + '_' + item.name"
                :rules="rules[item.name]"
                :prop="
                  `${namePath.length ? namePath.join('.') + '.' : ''}${
                    item.name
                  }`
                "
              >
                <!-- 非最后一个节点正常展示与渲染 -->
                <template v-if="item.slot === true">
                  <slot-content :name="item.name" :scope="value" />
                </template>

                <!-- 最后一个节点，需要带上 sortIndex 输入框 -->
                <template v-else-if="itemIndex === settings.length - 1">
                  <el-col style="visibility: hidden;width: 0;height: 0;">
                    <!-- <el-col span="0"> -->
                    <el-form-item
                      :prop="
                        `${
                          namePath.length ? namePath.join('.') + '.' : ''
                        }sortIndex`
                      "
                    >
                      <el-input v-model="value.sortIndex" />
                    </el-form-item>
                  </el-col>
                </template>

                <!-- 没有设置 slot 为 true 的统一用 input 框 -->
                <template v-else>
                  <el-input v-model="value[item.name]" />
                </template>
              </el-form-item>
            </el-col>

            <!-- 额外节点 -->
            <!-- <el-col :key="dataKey + '_' + item.name+ '_' + 'extra'">
              <slot :name="item.name + 'ExtraDom'" :scope="value[item.name]" />
            </el-col> -->
          </template>
        </el-col>

        <el-col class="relation-item-node-btn">
          <!-- 并且满足与删除 -->
          <el-col class="relation-op-btn">
            <template
              v-if="
                level === 1 || (namePath && namePath[namePath.length - 1] === 0)
              "
            >
              <el-tooltip content="新增配置" placement="top">
                <el-button
                  type="text"
                  size="small"
                  style="color: #1890ff"
                  @click="
                    e => {
                      handleTarget('add')
                    }
                  "
                >
                  并且满足
                </el-button>
              </el-tooltip>
            </template>

            <el-tooltip content="删除配置" placement="top">
              <el-button
                icon="el-icon-delete"
                type="text"
                size="small"
                style="color: red;"
                @click="
                  e => {
                    if (conditionsLen === 1) {
                      this.$message.warning('至少配置一条规则')
                      return
                    }
                    handleTarget('delete')
                  }
                "
              />
            </el-tooltip>
          </el-col>
        </el-col>
      </el-row>
    </template>
    <template v-if="conditionsLen">
      <!-- 且、或按钮  -->
      <div class="filter-content-line">
        <el-form-item
          :prop="`${namePath.length ? namePath.join('.') + '.' : ''}relation`"
        >
          <el-radio-group
            v-model="value.relation"
            class="relation-change-button"
            size="small"
            @change="handleRelation"
          >
            <el-radio-button label="and">且</el-radio-button>
            <el-radio-button label="or">或</el-radio-button>
          </el-radio-group>
        </el-form-item>
      </div>

      <!-- filters里面的数据渲染(传入conditions有多条时) -->
      <div style="paddingLeft: 40px">
        <relation-item
          v-for="(condItem, condIndex) in value.conditions"
          ref="relationForm"
          :key="level + '_' + condItem.sortIndex + '_' + condIndex"
          :data-key="level + '_' + condItem.sortIndex + '_' + condIndex"
          :value="condItem"
          :parent-value="value"
          :name-path="[...namePath, 'conditions', condIndex]"
          :level="level + 1"
          :settings="settings"
          :rules="rules"
          :item-default-value="itemDefaultValue"
          v-on="$listeners"
        />
      </div>
    </template>
  </div>
</template>

<script>
export default {
  name: 'RelationItem',
  components: {
    'slot-content': {
      props: {
        name: {
          required: true
        },
        scope: {
          required: true
        }
      },
      render(h) {
        const slot = this.$parent.elForm.$children[0].$scopedSlots
        return slot[this.name]({ scope: this.scope })
      }
    }
  },
  props: {
    // filter 里面的数据
    value: {
      // conditions or filters
      type: Object,
      default: () => ({})
    },
    parentValue: {
      type: Object,
      default: () => ({})
    },
    namePath: {
      type: Array,
      default: () => []
    },
    level: {
      type: Number,
      default: () => 0
    },
    settings: {
      type: Array,
      default: () => []
    },
    rules: {
      type: Object,
      default: () => ({})
    },
    itemDefaultValue: {
      type: Object,
      default: () => ({})
    },
    dataKey: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      childCondtionsMinLength: 2
    }
  },
  computed: {
    conditionsLen() {
      return this.value && this.value.conditions && this.value.conditions.length
    },
    slotKeys() {
      const slot = []
      this.settings.forEach(item => {
        if (item.slotBefore === true) {
          slot.push(`${item.name}DefaultBefore`)
        }
        if (item.type === 'slot' || item.slot === true) {
          slot.push(item.name)
        }
        if (item.slotAfter === true) {
          slot.push(`${item.name}DefaultAfter`)
        }
      })
      return slot
    }
  },
  methods: {
    /**
     * 并且满足或删除操作
     * @param action 操作类型
     */
    handleTarget(action) {
      const index = this.namePath && this.namePath[this.namePath.length - 1] // 选中当前项的索引

      if (action === 'add') {
        // 并且满足
        if (this.level === 1) {
          // 只有一条数据进行并且满足
          const firstData = { ...this.value } // 第一天数据，需要塞到conditions中，并把外面的数据删除
          this.$set(this.value, 'conditions', [
            firstData,
            { ...this.itemDefaultValue, sortIndex: this.value.sortIndex }
          ])

          this.$set(this.value, 'relation', 'and')

          Object.keys(this.value).map((key, index) => {
            if (!['conditions', 'relation', 'sortIndex'].includes(key)) {
              delete this.value[key]
            }
          })
        } else {
          // 多条数据进行并且满足
          this.parentValue.conditions.push({
            ...this.itemDefaultValue,
            sortIndex: this.value.sortIndex
          })
        }
      } else {
        // 删除
        if (!this.parentValue.conditions) {
          this.$message.warning('至少配置一条规则配置')
          return
        }

        if (
          this.parentValue.conditions.length === this.childCondtionsMinLength
        ) {
          // 有并且满足数据（只有两条）
          this.parentValue.conditions.splice(+index, 1)
          this.$nextTick(() => {
            this.settings.map(({ name }) => {
              this.$set(
                this.parentValue,
                name,
                this.parentValue.conditions[0][name]
              )
            })
            delete this.parentValue.conditions
            delete this.parentValue.relation
          })
        } else {
          // 超过两条，任意删除
          this.parentValue.conditions.splice(+index, 1)
        }
      }

      this.$emit('onChange', {
        type: action,
        value: this.value,
        namePath: this.namePath
      })
    },
    handleRelation(val) {
      this.$emit('onChange', {
        type: 'relation',
        val: val,
        value: this.value,
        namePath: this.namePath
      })
    }
  }
}
</script>

<style lang="scss">
.relational-form {
  .relation-change-button {
    display: inline-flex;
    flex-direction: column;
    position: relative;
    left: -16px;
    .el-radio-button__inner {
      padding: 7px 6.5px;
      box-shadow: 0px 0 0 0 $primary !important;
    }
    .el-radio-button:first-child .el-radio-button__inner {
      border-radius: 4px 4px 0 0;
      border-left: 1px solid #dcdfe6;
    }
    .el-radio-button:last-child .el-radio-button__inner {
      border-radius: 0 0 4px 4px;
      border-left: 1px solid #dcdfe6;
    }
  }
}
</style>

<style lang="scss" scoped>
.relation-item-node {
  position: relative;
  display: block;
  .relation-item-node-block {
    display: flex;
    .relation-item-node-base {
      flex: 1;
    }

    .relation-item-node-btn {
      width: 88px;
    }
  }
}

.filter-content-line {
  position: absolute;
  top: 0;
  height: calc(100% - 16px);
  display: flex;
  align-items: center;
  border-left: 1px solid #dcdfe6;
  .el-form-item {
    margin-bottom: 0;
  }
}

.relation-op-btn {
  padding-left: 5px;
  padding-right: 5px;
  margin-top: -3px;
}
</style>
