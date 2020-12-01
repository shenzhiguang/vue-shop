<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区域 -->
        <el-card class="box-card">
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addCate">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 树形表格 -->
            <tree-table class="tree-table"
              :data="cateList"
              :columns="columns"
              :selection-type="false"
              :expand-type="false"
              :show-index="true"
              index-text="#"
              :show-row-hover="false">
                <!-- 是否有效 -->
                <template slot="isOk" slot-scope="scope">
                    <i style="color: lightgreen" class="el-icon-success" v-if="scope.row.cat_deleted === false"></i>
                    <i style="color: red" class="el-icon-error" v-else></i>
                </template>
                <!-- 排序 -->
                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag size="mini" type="warning" v-else>三级</el-tag>
                </template>
                <!-- 操作 -->
                <template slot="opt" slot-scope="scope">
                    <el-button size="mini" type="primary" icon="el-icon-edit" @click="editCateBtn(scope.row)"></el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteCate(scope.row)"></el-button>
                </template>
            </tree-table>
            <!-- 分页区域 -->
            <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryInfo.pagenum"
              :page-sizes="[3, 5, 10, 15]"
              :page-size="queryInfo.pagesize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加分类对话框 -->
        <el-dialog
          title="添加分类"
          :visible.sync="addCateDialogVisible"
          width="50%"
          @close="addCateClose">
          <!-- 添加分类表单 -->
          <el-form :model="addCateForm" :rules="addCateRules" ref="addCateRef" label-width="100px" class="demo-ruleForm">
            <el-form-item label="分类名称" prop="cat_name">
              <el-input v-model="addCateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="分类层级">
              <!-- 级联选择器 -->
              <el-cascader style="width: 100%"
                v-model="selectDataValue"
                :options="selectData"
                :props="selectProps"
                @change="handleChange"
                :clearable="true"></el-cascader>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addCateConfirm">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 编辑对话框 -->
        <el-dialog
          title="修改分类"
          :visible.sync="editCateDialogVisible"
          width="50%"
          @close="editCateClose">
          <!-- 添加分类表单 -->
          <el-form :model="editCateForm" :rules="editCateRules" ref="editCateRef" label-width="100px" class="demo-ruleForm">
            <el-form-item label="分类名称" prop="cat_name">
              <el-input v-model="editCateForm.cat_name"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editCateConfirm">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      // 控制添加分类对话框的显示
      addCateDialogVisible: false,
      // 控制编辑对话框的显示
      editCateDialogVisible: false,
      // 添加分类表单数据
      addCateForm: {
        cat_name: '',
        // 分类父id, 默认0
        cat_pid: 0,
        // 分类层级，默认0
        cat_level: 0
      },
      // 编辑表单数据
      editCateForm: {
        cat_name: ''
      },
      // 添加分类表单验证规则
      addCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 编辑表单验证规则
      editCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 编辑提交数据的id
      editCateConifrmId: 0,
      // 级联选择器数据
      selectData: [],
      // 级联选择器配置
      selectProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 级联选择器选中的数据
      selectDataValue: [],

      // 请求商品分类数据参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 总商品分类数据条数
      total: 0,
      // 商品分类数据
      cateList: [],
      // 表格的每一列
      columns: [
        {
          label: '商品分类',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isOk'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ]
    }
  },
  created () {
    this.getCateList()
  },
  mounted () {
    setInterval(function () {
      document.querySelectorAll('.el-cascader-node__label').forEach(element => {
        element.onclick = function () {
          if (this.previousElementSibling) {
            this.previousElementSibling.click()
          }
        }
      })
    }, 1000)
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败！')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 分页
    handleSizeChange (size) {
      this.queryInfo.pagesize = size
      this.getCateList()
    },
    handleCurrentChange (num) {
      this.queryInfo.pagenum = num
      this.getCateList()
    },
    // 添加分类按钮
    async addCate () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类层级失败！')
      }
      this.selectData = res.data

      this.addCateDialogVisible = true
    },
    // 级联选择器选中
    handleChange (value) {
      this.addCateForm.cat_pid = value.length
      this.addCateForm.cat_level = value.length + 1
      this.selectDataValue = value
    },
    // 添加分类对话框关闭
    addCateClose () {
      this.$refs.addCateRef.resetFields()
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.selectDataValue = []
    },
    // 添加分类提交
    addCateConfirm () {
      this.$refs.addCateRef.validate(async bool => {
        if (!bool) {
          // eslint-disable-next-line no-useless-return
          return
        }
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败！')
        }
        this.$message.success('添加成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },

    // 编辑
    // 编辑分类按钮
    editCateBtn (cate) {
      this.editCateConifrmId = cate.cat_id
      this.editCateForm.cat_name = cate.cat_name
      this.editCateDialogVisible = true
    },
    // 编辑对话框关闭
    editCateClose () {
      this.$refs.editCateRef.resetFields()
    },
    // 编辑对话框提交
    editCateConfirm () {
      this.$refs.editCateRef.validate(async bool => {
        if (!bool) {
          // eslint-disable-next-line no-useless-return
          return
        }
        const { data: res } = await this.$http.put(`categories/${this.editCateConifrmId}`, this.editCateForm)
        if (res.meta.status !== 200) {
          return this.$message.error('修改失败！')
        }
        this.$message.success('修改成功！')
        this.getCateList()
        this.editCateDialogVisible = false
      })
    },
    // 删除
    deleteCate (cate) {
      this.$confirm('确定删除该项?', '删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`categories/${cate.cat_id}`)
        if (res.meta.status !== 200) { return this.$message.error('删除失败！') }
        this.queryInfo.pagenum = this.queryInfo.pagenum === Math.ceil(this.total / this.queryInfo.pagesize) ? Math.ceil(this.total / this.queryInfo.pagesize) : this.queryInfo.pagenum
        this.getCateList()
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.tree-table {
    margin: 15px 0;
}

</style>
