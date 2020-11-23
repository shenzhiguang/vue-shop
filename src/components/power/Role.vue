<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区域 -->
        <el-card class="box-card">
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表区域 -->
            <el-table
                :data="roleList"
                style="width: 100%"
                :stripe="true"
                :border="true">
                <el-table-column
                    type="expand">
                    <template v-slot:default="scope">
                        <el-row :class="['bor-bottom', 'vcenter', index1 === 0 ? 'bor-top' : '']" v-for="(item1, index1) in scope.row.children" :key="item1.id">
                            <!-- 一级权限 -->
                            <el-col :span="5">
                                <el-tag closable @close="deleteTag(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 二级和三级权限 -->
                            <el-col :span="19">
                                <el-row :class="['vcenter', index2 === 0 ? '' : 'bor-top']" v-for="(item2, index2) in item1.children" :key="item2.id">
                                    <!-- 二级权限 -->
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="deleteTag(scope.row, item2.id)">{{ item2.authName }}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <!-- 三级权限 -->
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="deleteTag(scope.row, item3.id)">{{ item3.authName }}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column
                    type="index"
                    label="#">
                </el-table-column>
                <el-table-column
                    prop="roleName"
                    label="角色名称">
                </el-table-column>
                <el-table-column
                    prop="roleDesc"
                    label="角色描述">
                </el-table-column>
                <el-table-column
                    label="操作">
                    <template v-slot:default="scope">
                        <el-button size="mini" type="primary" icon="el-icon-edit" @click="editRole(scope.row)"></el-button>
                        <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteRole(scope.row)"></el-button>
                        <el-tooltip :enterable="false" class="item" effect="dark" content="分配权限" placement="top">
                            <el-button size="mini" type="warning" icon="el-icon-setting" @click="setRight(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!-- 分配权限对话框 -->
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close="setCloseRight">
            <!-- 树形控件 -->
            <el-tree :data="roleRightList" :props="roleRightProps" :default-expand-all="true" show-checkbox node-key="id" :default-checked-keys="defKeys" ref="treeRef"></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="modifyRoleRight">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 添加角色对话框 -->
        <el-dialog
          title="添加角色"
          :visible.sync="addRoleDialogVisible"
          width="50%"
          @close="addRoleClose">
          <!-- 添加角色表单 -->
          <el-form ref="addRoleFormRef" :model="addRoleForm" :rules="addRoleFormRule" label-width="80px">
            <el-form-item label="角色名称" prop="roleName">
              <el-input v-model="addRoleForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
              <el-input v-model="addRoleForm.roleDesc"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addRoleConfirm">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 修改角色对话框 -->
        <el-dialog
          title="添加角色"
          :visible.sync="editRoleDialogVisible"
          width="50%"
          @close="editRoleClose">
          <!-- 修改角色表单 -->
          <el-form ref="addRoleFormRef" :model="addRoleForm" :rules="addRoleFormRule" label-width="80px">
            <el-form-item label="角色名称" prop="roleName">
              <el-input v-model="addRoleForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
              <el-input v-model="addRoleForm.roleDesc"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editRoleConfirm">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      // 角色列表数据
      roleList: [],
      // 控制分配权限对话框的显示
      setRightDialogVisible: false,
      // 控制添加角色对话框的显示
      addRoleDialogVisible: false,
      // 控制修改角色对话框的显示
      editRoleDialogVisible: false,
      // 分配角色权限列表
      roleRightList: [],
      // 定义树形控件的参数
      roleRightProps: {
        children: 'children',
        label: 'authName'
      },
      // 该角色所具有的权限
      defKeys: [],
      // 分配对话框对应的角色 id
      roleId: '',
      // 添加角色表单数据
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色表单数据验证
      addRoleFormRule: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 1, max: 10, message: '长度在 1 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          { min: 1, max: 20, message: '长度在 1 到 20 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    // 获取角色列表数据
    async getRoleList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.roleList = res.data
    },
    // 删除角色指定权限
    deleteTag (role, rightId) {
      this.$confirm('此操作将永久删除该权限, 是否继续?', '权限删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) {
          return this.$message.error('删除失败！')
        }
        role.children = res.data
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
    },
    // 分配权限
    async setRight (role) {
      // 获取角色的所有权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.roleRightList = res.data
      this.findRight(role, this.defKeys)
      this.roleId = role.id
      this.setRightDialogVisible = true
    },
    // 递归遍历出角色所具有的权限
    findRight (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(element => {
        this.findRight(element, arr)
      })
    },
    // 关闭对话框时清除数据
    setCloseRight () {
      this.defKeys = []
    },
    // 修改角色权限
    async modifyRoleRight () {
      const keyId = [...this.$refs.treeRef.getCheckedKeys(), this.$refs.treeRef.getHalfCheckedKeys()]
      const strKey = keyId.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: strKey
      })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.getRoleList()
      this.$message.success('分配权限成功！')
      this.setRightDialogVisible = false
    },
    // 关闭添加角色对话框，清空对话框中的数据
    addRoleClose () {
      this.$refs.addRoleFormRef.resetFields()
    },
    // 提交添加角色
    addRoleConfirm () {
      this.$refs.addRoleFormRef.validate(async bool => {
        // eslint-disable-next-line no-useless-return
        if (!bool) return
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败！')
        this.$message.success('添加角色成功！')
        this.getRoleList()
        this.addRoleDialogVisible = false
      })
    },
    // 编辑角色
    async editRole (role) {
      const { data: res } = await this.$http.get(`roles/${role.id}`)
      if (res.meta.status !== 200) return this.$message.error('获取角色失败！')
      this.addRoleForm = res.data
      this.editRoleDialogVisible = true
    },
    // 关闭修改角色对话框
    editRoleClose () {
      this.$refs.addRoleFormRef.resetFields()
      this.addRoleForm.roleName = ''
      this.addRoleForm.roleDesc = ''
      this.addRoleForm.roleId = ''
    },
    // 提交修改角色
    editRoleConfirm () {
      this.$refs.addRoleFormRef.validate(async bool => {
        // eslint-disable-next-line no-useless-return
        if (!bool) return
        const { data: res } = await this.$http.put(`roles/${this.addRoleForm.roleId}`, {
          roleName: this.addRoleForm.roleName,
          roleDesc: this.addRoleForm.roleDesc
        })
        if (res.meta.status !== 200) return this.$message.error('修改角色失败！')
        this.$message.success('修改角色成功！')
        this.getRoleList()
        this.editRoleDialogVisible = false
      })
    },
    // 删除角色
    deleteRole (role) {
      this.$confirm('确认删除该角色吗?', '删除角色', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${role.id}`)
        if (res.meta.status !== 200) return this.$message.error('删除失败！')
        this.getRoleList()
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
.el-table {
    margin-top: 15px;
}

.el-tag {
    margin: 7px;
}

.bor-bottom {
    border-bottom: 1px solid #eee;
}

.bor-top {
    border-top: 1px solid #eee;
}

.vcenter {
    display: flex;
    align-items: center;
}
</style>
