<!--
 * @Description: 格式化手机号公共组件
 * @Author: liushuling
 * @Date: 2024-06-18 15:26:48
 * @LastEditors: liushuling
 * @LastEditTime: 2024-06-18 17:31:37
-->
<template>
  <el-input
    v-model="content"
    type="textarea"
    clearable
    :rows="rows"
    :placeholder="placeholder"
    @input="handleInputMobiles"
  />
</template>

<script>
import { debounce } from '@/utils'

export default {
  name: 'FormatMobile',
  model: {
    prop: 'value',
    event: 'input'
  },
  props: {
    value: {
      type: String,
      default: ''
    },
    rows: {
      type: Number,
      default: 4
    },
    placeholder: {
      type: String,
      default: '请输入号码，多条数据使用回车分隔'
    }
  },
  data() {
    return {
      content: '',
      debounceInput: null
    }
  },
  watch: {
    value: {
      immediate: true,
      handler(val) {
        this.content = val
      }
    }
  },
  mounted() {
    this.debounceInput = debounce(
      val => this.formatMobiles(val), 100
    )
  },
  methods: {
    // 测试号码 input 框输入事件，支持自动换行
    handleInputMobiles(val) {
      return this.debounceInput(val)
    },
    // 格式化手机号
    formatMobiles(val) {
      let fVal = val.replace(/[\n\s,]/g, '') // 简化正则表达式，移除非数字字符
      fVal = fVal.replace(/[^\d]/g, '') // 确保只保留数字字符

      let newVal = '' // 初始化新值字符串

      // 循环处理，确保每11位后添加换行符，包括最后一个块
      for (let i = 0; i < fVal.length; i += 11) {
        newVal += fVal.substring(i, i + 11) + '\n' // 截取每11位并添加换行
      }

      // 移除最后一个多余的换行符（如果有的话）
      if (newVal.endsWith('\n')) {
        newVal = newVal.slice(0, -1)
      }

      this.$nextTick(() => {
        // 更新 data 中的 content 值
        this.content = newVal
        // 把格式化之后的手机号传给父组件
        this.$emit('input', newVal)
      })
    }
  }
}
</script>

