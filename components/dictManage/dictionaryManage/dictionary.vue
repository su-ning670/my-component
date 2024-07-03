<template>
  <!-- 字典 -->
  <div class="ucenter-container">
    <!--工具栏-->
    <el-card class="box-card head-container" shadow="never">
      <div slot="header" class="clearfix">
        <span> <svg-icon icon-class="search" />查询 </span>
      </div>
      <!-- 搜索 -->
      <el-row class="ucenter-search__form">
        <el-col :xs="24" :sm="6" :xl="6">
          <span class="search_label">系统名称：</span>
          <AppSelect
            v-model="localQuery.appId"
            @change="crud.toQuery"
            @get="(func) => (getAppList = func)"
          />
        </el-col>
        <el-col :xs="24" :sm="6" :xl="6">
          <span class="search_label">字典名称：</span>
          <el-input
            v-model="localQuery.blurry"
            size="mini"
            clearable
            placeholder="输入字典名称"
            style="width: 200px"
            class="filter-item"
            @keyup.enter.native="crud.toQuery"
          />
          <!-- <el-select v-model="localQuery.blurry" multiple placeholder="请选择字典名称">
            <el-option
              v-for="item in this.dictionaryList"
              :key="item.name"
              :label="item.name"
              :value="item.name"
            />
          </el-select> -->
        </el-col>
      </el-row>

      <el-row class="ucenter-search__btns">
        <!-- 新增删除 -->
        <crudOperation :permission="permission">
          <template slot="buttons">
            <el-button
              v-permission="['dictionaryManage:del']"
              class="filter-button"
              type="danger"
              icon="el-icon-delete"
              size="mini"
              :disabled="!crud.selections.length"
              @click="toDelete(crud.selections)"
            >删除</el-button>
            <el-button
              v-permission="['dict:sync']"
              type="success"
              icon="el-icon-refresh"
              :disabled="!localQuery.appId"
              @click="handleSync"
            >同步</el-button>
          </template>
        </crudOperation>
        <!-- 查询重置 -->
        <rrOperation />
      </el-row>
    </el-card>

    <!--字典管理表单渲染-->
    <el-dialog
      append-to-body
      :close-on-click-modal="false"
      :before-close="crud.cancelCU"
      :visible.sync="crud.status.cu > 0"
      :title="crud.status.title"
      class="ucenter-dialog__mini"
    >
      <el-form
        ref="form"
        :model="form"
        :rules="rules"
        size="mini"
        label-width="102px"
      >
        <el-form-item label="字典名称：" prop="name">
          <el-input
            v-model="form.name"
            placeholder="请输入字典名称"
            maxlength="50"
            :disabled="crud.status.title==='编辑字典'"
            show-word-limit
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
            maxlength="100"
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

    <!--新增字典属性表单渲染-->
    <el-dialog
      append-to-body
      :close-on-click-modal="false"
      :before-close="dictInfoCancelCU"
      :visible.sync="addDictInfoVisible"
      title="新增字典属性"
      class="ucenter-dialog__mini"
    >
      <el-form
        ref="addDictInfoForm"
        :model="addDictInfoForm"
        :rules="addDictInfoRules"
        size="mini"
        label-width="102px"
      >
        <el-form-item label="所属字典：" prop="dict">
          <el-select v-model="addDictInfoForm.dict" placeholder="请选择字典名称" disabled filterable>
            <el-option
              v-for="item in dictionaryList"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="键：" prop="label">
          <el-input
            v-model="addDictInfoForm.label"
            placeholder="请输入键名"
            maxlength="16"
            show-word-limit
          />
        </el-form-item>
        <el-form-item label="值：" prop="value">
          <el-input
            v-model="addDictInfoForm.value"
            placeholder="请输入具体值"
            maxlength="16"
            show-word-limit
          />
        </el-form-item>
        <el-form-item label="排序：" prop="dictSort">
          <el-input-number
            v-model.number="addDictInfoForm.dictSort"
            aria-placeholder="请选择排序"
            :min="0"
            :max="999"
            controls-position="right"
          />
        </el-form-item>
        <el-form-item label="启用状态：" prop="isEnable">
          <el-switch
            v-model="addDictInfoForm.isEnable"
            active-color="#13ce66"
          />
        </el-form-item>
        <el-form-item label="描述：" prop="description">
          <el-input
            v-model="addDictInfoForm.description"
            type="textarea"
            placeholder="请输入描述"
            maxlength="16"
            show-word-limit
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="text" @click="dictInfoCancelCU">取消</el-button>
        <el-button
          type="primary"
          :loading="crud.status.cu === 2"
          @click="dictInfoSubmitCU"
        >确认</el-button>
      </div>
    </el-dialog>

    <!-- 同步弹框（禁止点击遮罩关闭弹框） -->
    <el-dialog
      title="选择同步环境"
      :visible.sync="isSync"
      width="30%"
      :before-close="handleSyncClose"
      :close-on-click-modal="false"
    >
      <el-form :model="syncForm">
        <el-form-item required label="同步环境：" label-width="90px">
          <el-select v-model="syncForm.targetEnv" placeholder="请选择需要同步的环境">
            <el-option :disabled="this.$store.getters.appEnv==='qa'" label="测试" value="test" />
            <el-option :disabled="this.$store.getters.appEnv==='uat'" label="预发布" value="uat" />
            <el-option :disabled="this.$store.getters.appEnv==='prod'" label="生产" value="prod" />
          </el-select>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="hadleSyncCancel">取 消</el-button>
        <el-button type="primary" @click="hadleSyncSubmit">确 定</el-button>
      </span>
    </el-dialog>

    <!--表格渲染-->
    <el-card v-loading="crud.loading" class="box-card table-container" shadow="never">
      <div slot="header" class="clearfix">
        <span> <svg-icon icon-class="list" />字典列表 </span>
      </div>
      <el-table
        ref="table"
        highlight-current-row
        style="width: 100%"
        :data="crud.data"
        :max-height="minHeight"
        @selection-change="crud.selectionChangeHandler"
        @current-change="selectCurrentRow"
        @row-click="handleClickTableRow"
      >
        >
        <el-table-column
          type="selection"
          width="45"
        />
        <el-table-column prop="name" label="字典名称" />
        <el-table-column prop="description" label="描述" :show-overflow-tooltip="true" />
        <el-table-column prop="isEnable" label="启用状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.isEnable===true" type="success">开启</el-tag>
            <el-tag v-if="scope.row.isEnable===false" type="danger">关闭</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          v-permission="['dictionaryManage:edit', 'dictionaryManage:del']"
          label="操作"
          width="200px"
          align="center"
          fixed="right"
        >
          <template slot-scope="scope">
            <udOperation :data="scope.row" :permission="permission" :disabled-dle="!!scope.row.isEnable">
              <template slot="preButtons">
                <el-button
                  v-permission="['dictionaryManage:addInfo']"
                  type="primary"
                  @click.stop="handleAddDictInfo(scope.row)"
                >新增属性
                </el-button>
              </template>
            </udOperation>
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </el-card>
  </div>
</template>

<script>
import crudDictionary from '@/api/system/dictionaryManage'
import crudDictionaryInfo from '@/api/system/dictionaryDetails'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import pagination from '@crud/Pagination'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import AppSelect from '../components/AppSelect'
import { get as geCurrMap } from '@/api/system/app'

// crud交由presenter持有
const defaultForm = {
  name: null,
  isEnable: true,
  description: null
}

const defaultDictInfoForm = {
  label: null,
  value: null,
  dictSort: 999,
  isEnable: true,
  description: null
}

export default {
  name: 'Dictionary',
  components: {
    AppSelect,
    crudOperation, // 搜索栏新增删除按钮
    rrOperation, // 搜索与重置
    pagination, // 分页组件
    udOperation // 列表编辑删除按钮
  },
  cruds() {
    return CRUD({
      title: '字典',
      url: '/ams-api/api/dict', // 获取字典列表的 api
      crudMethod: { ...crudDictionary },
      optShow: {
        add: true,
        edit: false,
        del: false,
        download: true,
        reset: true
      },
      queryOnPresenterCreated: false
    })
  },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  props: ['shouldDeleteItem'], // 手动删除的字典项
  data() {
    // 字典名称校验规则
    const validateDictionName = (rule, value, callback) => {
      if (!value) {
        callback(new Error('请输入字典名称'))
        return
      } else {
        // 字母、数字、下滑线、美元符合组成
        const nameRegex = /^[a-zA-Z0-9_$]+$/

        if (!nameRegex.test(value)) {
          callback(new Error('仅允许字母、数字、下滑线、美元符合'))
        }
      }
      callback()
    }
    return {
      isSync: false, // 打开同步弹框的标识
      syncForm: {
        targetEnv: '' // 同步的环境的值
      },
      appList: [],
      localQuery: {}, // 当前搜索参数
      getAppList: undefined, // 获取系统列表
      dictionaryList: [], // 当前系统下的所有字典数据
      addDictInfoVisible: false, // 控制字典属性弹框是否展示
      addDictInfoForm: {}, // 字典属性弹框表单值
      addDictInfoFlag: 0, // 是否新增了字典属性，用于字典详情列表的刷新作用
      batchDeleteArr: [], // 存储勾选的字典项
      minHeight: window.screen.height > 1200 ? Math.ceil(window.innerHeight * 0.3 + 10) : Math.ceil(window.innerHeight * 0.2 + 10), // 动态设置 table 组件的高度
      permission: {
        add: ['dictionaryManage:add'],
        edit: ['dictionaryManage:edit'],
        del: ['dictionaryManage:del'],
        download: ['dictionaryManage:down']
      },
      rules: {
        name: [{ required: true, validator: validateDictionName, trigger: 'blur' }],
        isEnable: [{ required: true, message: '请选择是否启用', trigger: 'change' }],
        description: [{ required: false, message: '请输入描述', trigger: 'blur' }]
      },
      addDictInfoRules: {
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
    // 查询框的系统id监控
    'localQuery.appId'(val) {
      this.$emit('customEvent', {}, val) // 给父组件传递数据
    },
    // table 组件勾选值监控
    'crud.selections'(val) {
      this.$emit('customEvent', val, this.localQuery.appId) // 给父组件传递数据
    },
    // 手动删除的字典项
    'shouldDeleteItem'(val) {
      if (typeof (val) === 'number') {
        // 需要全部清空
        this.batchDeleteArr.splice(0, this.batchDeleteArr.length)

        this.$refs.table.clearSelection() // 清空所有选中
      } else {
        this.batchDeleteArr.map((item, index) => {
          if (item.name === val) {
            this.batchDeleteArr.splice(index, 1)
            this.$refs.table.toggleRowSelection(item)
          }
        })
      }
    }
    // 查询框的字典名称监控
    // 'localQuery.blurry'(val) {
    //   this.$emit('customEvent', {}, val) // 给父组件传递数据
    // }
  },
  mounted() {
    this.$emit('customEvent', {}, this.localQuery.appId) // 给父组件传递数据

    this.crud.loading = true

    this.getAppList &&
      this.getAppList()
        .then((data) => {
          if (!data) return
          const app = data.find((app) => app.sysId === this.$appId)

          this.localQuery = {
            ...this.localQuery,
            appId: app ? app.id : data[0].id
          }
          this.queryOnPresenterCreated = true
          this.appList = data
          this.crud.toQuery()

          // 查询所有的字典列表
          crudDictionary.getDict({ appId: app.id }).then(res => {
            this.dictionaryList = res.content // 查询所有的字典列表数据，用于后面的新增查重使用
          })
        })
        .catch(() => {
          this.crud.loading = false
        })
  },
  methods: {
    // 重置前的操作（清空查询框的字典名称，appid 不清空）
    [CRUD.HOOK.beforeResetQuery]() {
      this.localQuery = {
        ...this.localQuery,
        blurry: ''
      }
    },
    // 刷新前的操作（把系统标识塞进查询参数中）
    [CRUD.HOOK.beforeRefresh]() {
      this.crud.query = {
        ...this.localQuery,
        appId: [this.localQuery.appId]
      }
      this.$emit('dictQuery', this.crud.query) // 给父组件传递数据
    },
    // 刷新后的操作
    [CRUD.HOOK.afterRefresh]() {
      // if (this.localQuery.blurry.length > 0) {
      //   // 如果字典名称也不为空需要把选中的字典名称全部给字典详情组件查询
      // }
    },
    // 新增与编辑前的操作
    [CRUD.HOOK.beforeToCU]({ form }) {
    },
    // 新增与编辑弹框显示后做的操作
    [CRUD.HOOK.afterToCU](crud, form) {
      // 如果是新增把 id 删除
      if (this.crud.status.title === '新增字典') {
        delete form.id
      }
      form.app = {
        id: this.localQuery.appId
      }
    },
    // 提交前的操作
    [CRUD.HOOK.beforeSubmit]({ form }) {
      // 字典名称查重
      if (this.crud.status.title === '新增字典' && this.dictionaryList.map(i => (i.name)).includes(form.name)) {
        this.$message.warning('字典名称重复请重新输入')
        return false
      } else {
        // 正常提交
      }
    },
    /**
     * 选中 table 的某一行
     */
    selectCurrentRow(val) {
      if (!val) {
        this.$emit('customEvent', {}, this.localQuery.appId) // 给父组件传递数据
      } else {
        this.crud.selections.push(val)
        // this.$emit('customEvent', val, this.localQuery.appId) // 给父组件传递数据
      }
    },
    /**
     * 新增字典属性
     */
    handleAddDictInfo(row) {
      this.addDictInfoVisible = true
      this.addDictInfoForm = {
        ...defaultDictInfoForm,
        dict: row.id
      }
    },
    /**
     * 关闭字典详情弹框
     */
    dictInfoCancelCU() {
      this.addDictInfoVisible = false
    },
    /**
     * 提交字典详情弹框
     */
    dictInfoSubmitCU() {
      this.$refs.addDictInfoForm.validate((valid) => {
        if (valid) {
          // 调用接口保存
          const addValue = {
            ...this.addDictInfoForm,
            dict: {
              id: this.addDictInfoForm.dict
            }
          }
          crudDictionaryInfo.add(addValue).then(res => {
            this.$message.success('字典属性新增成功')
            this.addDictInfoFlag += 1
            this.addDictInfoVisible = false
            this.$emit('addDictFlag', this.addDictInfoFlag) // 给父组件传递新增标识，用于兄弟组件的刷新
          })
        } else {
          // this.$message.error('请完善表单相关信息！')
          return false
        }
      })
    },
    /**
     * 实现 table 组件点击行勾选某一行数据的功能
     */
    handleClickTableRow(row, event, column) {
    // checkNum和isCheck是批量操作按钮和已选几条展示的依据
      this.checkNum = 0
      this.isCheck = false
      // 如果status为无效不让勾选
      if (row.status === '0') {
        return
      } else {
        // 选中这行数据 勾选框会勾选上
        this.$refs.table.toggleRowSelection(row)
        if (this.batchDeleteArr.length > 0) {
          // 如果结果数组不为空，判断所选的这条是否在结果数组里
          if (JSON.stringify(this.batchDeleteArr).indexOf(JSON.stringify(row)) === -1) {
            this.batchDeleteArr.push(row)
          } else {
            // 所选数据在结果数组里，那就要清除掉它，不然会造成数据重复
            this.batchDeleteArr.map((item, index) => {
              if (item.id === row.id) {
                this.batchDeleteArr.splice(index, 1)
              }
            })
          }
        } else {
          this.batchDeleteArr.push(row)
        }
      }
      // 这里几行代码是根据结果数组是否有值判断批量操作按钮和已选几条展示是否需要解除置灰和展示
      if (this.batchDeleteArr.length > 0) {
        this.checkNum = this.batchDeleteArr.length
        this.isCheck = true
      } else {
        this.isCheck = false
      }
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

          this.$confirm(`此操作将永久删除字典名称【${falseData.map(i => i.name).join(',')}】，其中状态已开启的字典不能删除，是否继续？`, '提示', {
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
          this.$confirm(`此操作将永久删除字典名称【${datas.map(i => i.name).join(',')}】，是否继续？`, '提示', {
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
        this.$message.warning('所选字典状态已启用不能删除')
      }
    },
    checkSelections() {
      if (!this.crud.selections.length) {
        this.$message.error('请先勾选字典')
        return false
      }
      return true
    },
    // 同步按钮，点击打开弹框
    handleSync() {
      this.isSync = true
    },
    // 关闭同步弹框
    handleSyncClose(done) {
      done()
      this.syncForm = { // 清空表单
        targetEnv: ''
      }
    },
    // 字典同步取消
    hadleSyncCancel() {
      this.isSync = false
      this.syncForm = { // 清空表单
        targetEnv: ''
      }
    },
    // 字典同步提交
    hadleSyncSubmit() {
      if (!this.checkSelections()) {
        return
      }

      geCurrMap({ blurry: this.appName }).then((data) => {
      // data 是rbac系统的所有信息，需要它的 sysId 值
        if (!data || !data.content) {
          this.$message.error('获取系统信息失败')
          return
        }
        const systemId = data.content[0].sysId // 获取选择系统的 sysId
        crudDictionary.syncDict(
          this.crud.selections.map(i => i.id),
          this.syncForm.targetEnv,
          systemId
        ).then(res => {
          // if (!res.success) return
          this.isSync = false // 关闭弹框
          this.syncForm = { // 清空表单
            targetEnv: ''
          }
          this.crud.toQuery() // 清空勾选
          this.$message.success('同步成功')
        })
      })
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
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
