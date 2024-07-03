<!--
 * @Description: 文件描述
 * @Author: liushuling
 * @Date: 2023-07-28 11:25:26
 * @LastEditors: liushuling
 * @LastEditTime: 2023-08-21 18:30:23
-->
<template>
  <div v-loading="crud.loading">
    <!-- 字典列表 -->
    <DictionaryList :should-delete-item="shouldDeleteItem" @customEvent="handleDataFromDict" @addDictFlag="handleAddDictFlag" @dictQuery="handleQuery" />
    <!-- 字典详情列表 -->
    <DictionaryInfo :dict-row-detail="toDictInfoData" :app-id="appId" :add-dict-flag="addDictFlag" :dict-curr-query="dictQueryData" @customDelete="handleDelete" />
  </div>
</template>

<script>
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import CRUD, { presenter, header, crud } from '@crud/crud'
import DictionaryList from './dictionary.vue'
import DictionaryInfo from './dictionaryInfo.vue'

export default {
  name: 'DictionaryManage',
  components: {
    DictionaryList,
    DictionaryInfo
  },
  cruds() {
    return CRUD({
    })
  },
  mixins: [presenter(), header(), crud()],
  data() {
    return {
      toDictInfoData: null, // 需要传给 DictInfo 组件中的数据
      appId: null, // 需要传给 DictInfo 组件中的appid
      addDictFlag: 0, // 需要传给 DictInfo 组件中用于刷新组件的标识
      shouldDeleteItem: '', // 需要传给 Dict 组件中需要取消选中的字典
      dictQueryData: {} // 字典组件的查询参数
    }
  },
  computed: {
  },
  watch: {
  },
  mounted() {
  },
  methods: {
    /**
     * 字典组件选中的某一行
     * data 某一行数据
     * appId 系统 id
     */
    handleDataFromDict(data, appId) {
      this.toDictInfoData = data
      this.appId = appId
    },
    /**
     * 字典属性新增标识
     */
    handleAddDictFlag(val) {
      this.addDictFlag = val
    },
    /**
     * 字典详情中手动删除的某一个字典
     */
    handleDelete(val) {
      this.shouldDeleteItem = val
    },
    /**
     * 字典组件的参数参数
     */
    handleQuery(val) {
      this.dictQueryData = val
    }
  }
}
</script>

