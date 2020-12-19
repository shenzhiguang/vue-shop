<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片区域 -->
        <el-card class="box-card">
            <el-alert
                title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                show-icon
                :closable="false">
            </el-alert>
            <el-row class="rowcol">
                <el-col>
                    <span>选择商品分类：</span>
                    <!-- 级联选择器 -->
                    <el-cascader
                        v-model="selectDataValue"
                        :options="cateList"
                        :props="selectProps"
                        @change="handleChange"
                        :clearable="true"></el-cascader>
                </el-col>
            </el-row>
            <!-- Tab 栏切换 -->
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
              <!-- 添加动态参数面板 -->
              <el-tab-pane label="动态参数" name="many">
                <el-button
                  type="primary"
                  size="mini"
                  :disabled="selectDataValue.length === 0"
                  @click="addParams"
                >添加参数</el-button>
                <!-- 动态参数表格 -->
                <el-table
                  border
                  stripe
                  :data="manyTableData"
                  style="width: 100%"
                  empty-text=" ">
                  <!-- 展开行 -->
                  <el-table-column type="expand"></el-table-column>
                  <el-table-column type="index" label="#"></el-table-column>
                  <el-table-column
                    prop="attr_name"
                    label="参数名称">
                  </el-table-column>
                  <el-table-column
                    label="操作">
                    <template>
                      <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
                      <el-button size="mini" type="danger" icon="el-icon-delete"></el-button>
                    </template>
                  </el-table-column>
                </el-table>
              </el-tab-pane>
              <!-- 添加静态属性面板 -->
              <el-tab-pane label="静态属性" name="only">
                <el-button
                  type="primary"
                  size="mini"
                  :disabled="selectDataValue.length === 0"
                  @click="addParams"
                >添加属性</el-button>
                <!-- 静态属性表格 -->
                <el-table
                  border
                  stripe
                  :data="onlyTableData"
                  style="width: 100%"
                  empty-text=" ">
                  <!-- 展开行 -->
                  <el-table-column type="expand"></el-table-column>
                  <el-table-column type="index" label="#"></el-table-column>
                  <el-table-column
                    prop="attr_name"
                    label="属性名称">
                  </el-table-column>
                  <el-table-column
                    label="操作">
                    <template>
                      <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
                      <el-button size="mini" type="danger" icon="el-icon-delete"></el-button>
                    </template>
                  </el-table-column>
                </el-table>
              </el-tab-pane>
            </el-tabs>
        </el-card>
        <!-- 添加参数 / 属性 对话框 -->
        <el-dialog
          :title="activeName === 'many' ? '添加参数' : '添加属性'"
          :visible.sync="addParamsDialogVisible"
          width="50%"
          @close="closeAddForm">
          <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
            <el-form-item :label="paramsOr" prop="attr_name">
              <el-input v-model="addForm.attr_name"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addParamsDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParamsSend">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    // 自定义验证
    var repeatOr = (rule, data, callback) => {
      let items = this.onlyTableData
      if (this.activeName === 'many') {
        items = this.manyTableData
      }
      for (const item of items) {
        if (item.attr_name === data) return callback(new Error('已存在该名称，请更换一个'))
      }
    }
    return {
      // 商品分类数据
      cateList: [],
      // 级联选择器配置项
      selectProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的数据
      selectDataValue: [],
      // Tab 栏切换激活的栏目
      activeName: 'many',
      // 动态参数数据
      manyTableData: [],
      // 静态属性数据
      onlyTableData: [],
      // 控制添加参数 / 属性 对话框的显示
      addParamsDialogVisible: false,
      // 添加对话框中的表单数据
      addForm: {
        attr_name: ''
      },
      // 添加对话框中的表单数据验证规则
      addFormRules: {
        attr_name: [
          { required: true, message: '名称不能为空', trigger: 'blur' },
          { validator: repeatOr, trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  computed: {
    paramsOr () {
      return this.activeName === 'many' ? '参数名称' : '属性名称'
    }
  },
  methods: {
    // 获取商品分类数据列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据列表失败！')
      }
      this.cateList = res.data
      // console.log(res.data)
    },
    // 级联选择器选中
    async handleChange () {
      this.manyTableData = []
      this.onlyTableData = []
      this.getParamsData()
      // console.log(this.selectDataValue)
    },
    // 获取参数列表数据
    async getParamsData () {
      // 选中发请求，根据所选分类的 id 获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.selectDataValue[2]}/attributes`, { params: { sel: this.activeName } })
      // console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 点击 Tab 栏切换事件
    handleTabClick () {
      // 优化 在有manyTableData 和 onlyTableData 的时候不再发起请求
      (this.manyTableData && this.manyTableData.length && this.onlyTableData && this.onlyTableData.length) || this.getParamsData()
      // console.log(this.activeName)
      // console.log(this.onlyTableData)
      // console.log(this.manyTableData)
      // console.log(!!this.onlyTableData)
    },
    // 点击添加参数 / 属性 按钮
    addParams () {
      this.addParamsDialogVisible = true
    },
    // 关闭对话框
    closeAddForm () {
      this.$refs.addFormRef.resetFields()
    },
    // 添加参数 / 属性 提交
    addParamsSend () {
      this.$refs.addFormRef.validate(async bool => {
        // eslint-disable-next-line no-useless-return
        if (!bool) return
        const { data: res } = await this.$http.post(`categories/${this.selectDataValue[2]}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败！')
        }
        this.getParamsData()
        this.$message.success('添加成功！')
        this.addParamsDialogVisible = false
        // console.log(res)
      })
    }
  }
}
</script>

<style lang="less" scoped>
.rowcol {
    margin: 15px 0;
}
.el-table {
  margin-top: 15px;
}
</style>
