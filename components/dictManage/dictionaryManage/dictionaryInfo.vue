<template>
  <!-- 字典详情 -->
  <div class="ucenter-container">
    <!--工具栏-->
    <!-- 搜索 -->
    <el-row class="dict-title">
      字典属性
      <template v-if=" (dictRowDetail && dictRowDetail.length > 0) ">
        已选中：
        <el-tag
          v-for="( item, index ) in tipDitcName"
          :key="item + index"
          :closable="item !== undefined"
          class="dict-select"
          @close="handleClose(item)"
        >
          {{ item && item }}
        </el-tag>
      </template>
      <template v-if=" (dictRowDetail && dictRowDetail.length > 0) ">
        <el-tag
          key="reset"
          type="danger"
          class="dict-reset"
          @click="handleRest"
        >
          清空
        </el-tag>
      </template>
    </el-row>
    <el-row class="ucenter-search__form" style="paddingBottom: 20px">
      <el-col :xs="24" :sm="6" :xl="6">
        <el-input
          v-model="localQuery.label"
          size="mini"
          clearable
          placeholder="输入键名"
          style="paddingRight: 10px"
          class="filter-item"
          @keyup.enter.native="crud.toQuery"
        />
      </el-col>
      <el-col :xs="24" :sm="6" :xl="6">
        <el-input
          v-model="localQuery.value"
          size="mini"
          clearable
          placeholder="输入具体值"
          class="filter-item"
          @keyup.enter.native="crud.toQuery"
        />
      </el-col>

      <!-- 查询重置 -->
      <rrOperation style="float: left ;marginLeft: 13px" />
      <!-- 新增删除 -->
      <crudOperation :permission="permission" style="float: left; marginLeft: 13px">
        <template slot="buttons">
          <el-button
            v-permission="['dictionaryInfo:del']"
            class="filter-button"
            type="danger"
            icon="el-icon-delete"
            size="mini"
            :disabled="!crud.selections.length"
            @click="toDelete(crud.selections)"
          >删除</el-button>
        </template>
      </crudOperation>
    </el-row>

    <!--字典详情管理表单渲染-->
    <el-dialog
      append-to-body
      :close-on-click-modal="false"
      :before-close="crud.cancelCU"
      :visible.sync="crud.status.cu > 0"
      :title="crud.status.title"
      class="ucenter-dialog__mini"
      @open="handleOpen"
    >
      <el-form
        ref="form"
        :model="form"
        :rules="rules"
        size="mini"
        label-width="102px"
      >
        <el-form-item label="所属字典：" prop="dict">
          <el-select
            v-model="form.dict"
            placeholder="请选择字典名称"
            filterable
            remote
            :remote-method="remoteMethod"
            :disabled="crud.status.title==='编辑字典详情'"
          >
            <el-option
              v-for="item in dictionaryDynaList"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="键：" prop="label">
          <el-input
            v-model="form.label"
            placeholder="请输入键名"
            maxlength="128"
            :disabled="crud.status.title==='编辑字典详情'"
            show-word-limit
          />
        </el-form-item>
        <el-form-item label="值：" prop="value">
          <el-input
            v-model="form.value"
            placeholder="请输入具体值"
            maxlength="128"
            show-word-limit
          />
        </el-form-item>
        <el-form-item label="排序：" prop="dictSort">
          <el-input-number
            v-model.number="form.dictSort"
            aria-placeholder="请选择排序"
            :min="0"
            :max="999"
            controls-position="right"
          />
        </el-form-item>
        <el-form-item label="启用状态：" prop="isEnable">
          <el-switch
            v-model="form.isEnable"
            active-color="#13ce66"
          />
        </el-form-item>
        <el-form-item label="描述：" prop="description">
          <el-input
            v-model="form.description"
            type="textarea"
            placeholder="请输入描述"
            maxlength="16"
            show-word-limit
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="text" @click="crud.cancelCU">取消</el-button>
        <el-button
          type="primary"
          :loading="crud.status.cu === 2"
          @click="crud.submitCU"
        >确认</el-button>
      </div>
    </el-dialog>

    <!--表格渲染-->
    <el-card v-loading="crud.loading" class="box-card table-container" shadow="never">
      <div slot="header" class="clearfix">
        <span> <svg-icon icon-class="list" />字典详情列表 </span>
      </div>
      <el-table
        ref="table"
        highlight-current-row
        style="width: 100%"
        :max-height="minHeight"
        :data="crud.data"
        @selection-change="crud.selectionChangeHandler"
      >
        <el-table-column
          type="selection"
          width="45"
        />
        <el-table-column prop="belongs" label="所属字典">
          <template slot-scope="scope">
            {{ scope.row.dict.name }}
          </template>
        </el-table-column>
        <el-table-column prop="label" label="键" />
        <el-table-column prop="value" label="值" />
        <el-table-column prop="dictSort" label="排序" />
        <el-table-column prop="description" label="描述" :show-overflow-tooltip="true" />
        <el-table-column prop="isEnable" label="启用状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.isEnable===true" type="success">开启</el-tag>
            <el-tag v-if="scope.row.isEnable===false" type="danger">关闭</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          v-permission="['dictionaryInfo:edit', 'dictionaryInfo:del']"
          label="操作"
          width="150px"
          align="center"
          fixed="right"
        >
          <template slot-scope="scope">
            <udOperation :data="scope.row" :permission="permission" :disabled-dle="!!scope.row.isEnable" />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </el-card>
  </div>
</template>

<script>
import crudDictionary from '@/api/system/dictionaryDetails'
import crudDictionaryMange from '@/api/system/dictionaryManage'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import pagination from '@crud/Pagination'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'

// crud交由presenter持有
const defaultForm = {
  dict: null,
  label: null,
  value: null,
  dictSort: 999,
  isEnable: true,
  description: null
}

export default {
  name: 'DictionaryInfo',
  components: {
    crudOperation, // 搜索栏新增删除按钮
    rrOperation, // 搜索与重置
    pagination, // 分页组件
    udOperation // 列表编辑删除按钮
  },
  cruds() {
    return CRUD({
      title: '字典详情',
      url: '/ams-api/api/dictDetail', // 获取字典详情列表的 api
      crudMethod: { ...crudDictionary },
      optShow: {
        add: true,
        // add: (Object.keys(this.dictRowDetail).length !== 0 && this.dictRowDetail && this.tipDitcName !== undefined) && !!this.dictRowDetail,
        edit: false,
        del: false,
        download: false,
        reset: true
      },
      queryOnPresenterCreated: false
    })
  },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  props: ['dictRowDetail', 'appId', 'addDictFlag', 'dictCurrQuery'], // 字典组件某一行的数据和系统标识
  data() {
    return {
      localQuery: {}, // 当前搜索参数
      tipDitcName: [], // 用于展示的字典名称
      dictionaryList: [], // 当前系统下的所有字典数据
      dictionaryDynaList: [], // 当前系统下用于远程搜索所有字典数据
      minHeight: window.screen.height > 1200 ? Math.ceil(window.innerHeight * 0.2 + 100) : Math.ceil(window.innerHeight * 0.1 + 100), // 动态设置 table 组件的高度
      resetNum: 0, // 用于清空所选字典的标识
      permission: {
        add: ['dictionaryInfo:add'],
        edit: ['dictionaryInfo:edit'],
        del: ['dictionaryInfo:del']
      },
      rules: {
        dict: [{ required: true, message: '请选择所属字典', trigger: 'blur' }],
        label: [{ required: true, message: '请输入键名', trigger: 'blur' }],
        value: [{ required: true, message: '请输入具体值', trigger: 'blur' }],
        dictSort: [{ required: true, message: '请输入排序', trigger: 'blur' }],
        isEnable: [{ required: true, message: '请选择是否启用', trigger: 'change' }],
        description: [{ required: false, message: '请输入描述', trigger: 'blur' }]
      }
    }
  },
  computed: {
    appName() {
      const app = this.appList.find(app => app.id === this.localQuery.appId)
      return app ? app.name : ''
    }
  },
  watch: {
    'dictRowDetail'(val) {
      if (Object.keys(val).length !== 0) {
        this.tipDitcName = val.map(i => i.name) // 装所有点击了的字典名称的集合
      }
      if (!this.appId) return
      this.localQuery = {
        ...this.localQuery,
        appId: this.appId,
        dictNames: Object.keys(val).length !== 0 ? val.map(i => i.name).join(',') : '',
        label: '', // 清空搜索栏里面的键
        value: '' // 清空搜索栏里面的值
      }
      this.crud.loading = true
      val && crudDictionary.getDictDetail({ dictNames: Object.keys(val).length !== 0 ? val.map(i => i.name).join(',') : '', appId: this.appId, page: 0, size: 20 }).then(res => {
        this.crud.data = res.content
        this.crud.page.total = res.totalElements || 0

        this.crud.loading = false
      })
    },
    'appId'(val) {
      if (!val) return
      this.localQuery = {
        ...this.localQuery,
        appId: this.appId,
        dictNames: '',
        label: '', // 清空搜索栏里面的键
        value: '' // 清空搜索栏里面的值
      }
      this.crud.loading = true
      val && crudDictionary.getDictDetail({ appId: val, page: 0, size: 20 }).then(res => {
        this.crud.data = res.content
        this.crud.page.total = res.totalElements || 0

        this.crud.loading = false
      })

      // 查询所有的字典列表
      this.getDictList()
    },
    'addDictFlag'(val) {
      // 新增字典属性的标识，如果发生变化了就刷新组件
      if (!val) return
      if (val !== 0) {
        this.crud.toQuery()
      }
    },
    'dictCurrQuery'(val) {
      this.crud.loading = true
      val && crudDictionary.getDictDetail({ dictNames: val.blurry ? val.blurry : '', appId: this.appId, page: 0, size: 20 }).then(res => {
        this.crud.data = res.content
        this.crud.page.total = res.totalElements || 0

        this.crud.loading = false
      })
    }
  },
  mounted() {
    this.crud.loading = true
  },
  methods: {
    // 重置前的操作（清空键和值）
    [CRUD.HOOK.beforeResetQuery]() {
      this.localQuery = {
        ...this.localQuery,
        label: '',
        value: ''
      }
    },
    // 刷新前的操作（把系统标识塞进查询参数中）
    [CRUD.HOOK.beforeRefresh]() {
      this.crud.query = {
        ...this.localQuery,
        appId: [this.localQuery.appId]
      }

      // 清空字典名称的查询
      if (this.tipDitcName.length === 0) {
        this.crud.query = {
          ...this.localQuery,
          appId: [this.localQuery.appId],
          dictNames: ''
        }
      }
    },
    // 新增与编辑前的操作（把选中的某个字典的 id 带到 form 中）
    [CRUD.HOOK.beforeToCU]({ form }) {
      // if (!this.dictRowDetail.name) {
      //   this.$message.warning('请先选中某条字典再添加字典详情')
      //   return false
      // }
      if (form.dict && form.dict.name) {
        // 编辑
        form.dict = form.dict.id // 具体某个字典的 Id，用于回显所属字典
      }
    },
    // 新增与编辑弹框显示后做的操作
    [CRUD.HOOK.afterToCU](crud, form) {
      // 如果是新增把 id 删除
      if (this.crud.status.title === '新增字典详情') {
        delete form.id
      }
    },
    // 提交前的操作
    [CRUD.HOOK.beforeSubmit]({ form }) {
      // if (!form.dict.id || form.dict.id === undefined) {
      //   this.$message.warning('请先选中某条字典再添加字典详情')
      //   return false
      // }
      if (this.crud.status.title === '新增字典详情') {
        form.dict = {
          id: form.dict
        }
      }
      if (this.crud.status.title === '编辑字典详情') {
        form.dict = {
          id: form.dict
        }
      }
    },
    // 编辑之前的操作
    [CRUD.HOOK.beforeToEdit](crud, form) {
      this.crud.toQuery()
    },
    /**
     * 清除 tag 标签
     */
    handleClose(val) {
      const index = this.tipDitcName.indexOf(val)

      this.dictRowDetail.splice(index, 1)
      this.tipDitcName.splice(index, 1)

      this.crud.query = {
        ...this.localQuery,
        appId: [this.localQuery.appId],
        dictNames: this.tipDitcName.join(','),
        label: '',
        value: ''
      }

      this.$emit('customDelete', val) // 给父组件传递数据

      // this.crud.toQuery()
    },
    /**
     * 手动清空所选的字典项
     */
    handleRest() {
      this.resetNum += 1

      this.dictRowDetail.splice(0, this.dictRowDetail.length)

      this.$emit('customDelete', this.resetNum) // 给父组件传递数据
    },
    /**
     * 多删
     */
    toDelete(datas) {
      if (datas.findIndex(i => i.isEnable === false) !== -1) {
        // 有关闭的数据（只传递关闭状态数据的 Id）
        if (datas.map(i => i.isEnable).indexOf(true) !== -1) {
          // 有开有关（只传递关闭状态数据的 Id）
          const falseData = [] // 存储关闭状态的数据
          datas.map((o) => {
            if (o.isEnable === false) {
              falseData.push(o)
            }
          })
          this.$confirm(`此操作将永久删除键【${falseData.map(i => i.label + '(' + (i.dict.name) + ')').join(',')}】，其中状态已启用的键不能删除，是否继续？`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          })
            .then(() => {
              this.crud.delAllLoading = true
              this.crud.doDelete(falseData)
            })
            .catch(() => {})
        } else {
          // 全关
          this.$confirm(`此操作将永久删除键【${datas.map(i => i.label + '(' + (i.dict.name) + ')').join(',')}】，是否继续?`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          })
            .then(() => {
              this.crud.delAllLoading = true
              this.crud.doDelete(datas)
            })
            .catch(() => {})
        }
      } else {
        // 全开
        this.$message.warning('所选键名的状态已启用不能删除')
      }
    },
    /**
     * 每次打开弹框都要重新拉取一下所有的字典列表
     */
    getDictList() {
      // 查询所有的字典列表
      crudDictionaryMange.getDict({ appId: this.appId, size: 10000 }).then(res => {
        this.dictionaryList = res.content // 查询所有的字典列表数据，用于后面的新增查重使用
      })
    },
    /**
     * 远程搜索
     */
    remoteMethod(query) {
      if (query !== '') {
        this.loading = true
        this.getDictList() // 远程搜索

        setTimeout(() => {
          this.loading = false
          this.dictionaryDynaList = this.dictionaryList.filter(item => { // 搜索条件过滤
            return item.name.toLowerCase()
              .indexOf(query.toLowerCase()) > -1
          })
        }, 200)
      } else {
        this.dictionaryDynaList = this.dictionaryList || [] // 搜索为空时，显示所有数据
      }
    },
    /**
     * 打开弹框
     * 每次打开弹框都重新赋值所有的字典数据
     */
    handleOpen() {
      this.dictionaryDynaList = this.dictionaryList || []
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.dict-title {
  font-weight: 700;
  font-size: 14px;
  padding-bottom: 20px;
  .dict-select {
    font-weight: 400;
    margin-right: 5px;
  }
  .dict-reset {
     font-weight: 400;
    margin-right: 5px;
    &:hover{
      cursor: pointer;
    }
  }
}
::v-deep .el-input-number .el-input__inner {
  text-align: left;
}
::v-deep .el-form .el-form-item__label {
  white-space: nowrap;
}
::v-deep .icon-close {
  position: relative;
  top: 2px;
  font-size: 16px;
  cursor: pointer;
  &:hover {
    color: $menuActiveText;
  }
}
::v-deep .text-pretend {
  textarea {
    text-indent: 300px;
  }
}
.text-pretend:before {
  background-color: #f5f7fa;
  color: #909399;
  display: block;
  position: absolute;
  content: attr(data-before);
  top: 4px;
  left: 12px;
  border-radius: 4px;
  padding: 4px;
  white-space: nowrap;
  line-height: normal;
}

.el-select .el-input {
  width: 130px;
}
.input-with-select .el-input-group__prepend {
  background-color: #fff;
}
</style>
