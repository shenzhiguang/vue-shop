<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片区域 -->
        <el-card class="box-card">
            <el-table
                :data="rightList"
                style="width: 100%"
                :stripe="true"
                :border="true">
                <el-table-column
                    type="index"
                    label="#">
                </el-table-column>
                <el-table-column
                    prop="authName"
                    label="权限名称">
                </el-table-column>
                <el-table-column
                    prop="path"
                    label="路径">
                </el-table-column>
                <el-table-column
                    prop="level"
                    label="权限等级">
                    <template v-slot:default="scope">
                        <el-tag v-if="scope.row.level === '0'">一级</el-tag>
                        <el-tag v-if="scope.row.level === '1'" type="success">二级</el-tag>
                        <el-tag v-if="scope.row.level === '2'" type="warning">三级</el-tag>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
    </div>
</template>

<script>
export default {
  data () {
    return {
      rightList: [{
        authName: '',
        path: '',
        level: 0
      }]
    }
  },
  created () {
    this.getRightList()
  },
  methods: {
    async getRightList () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.rightList = res.data
    }
  }
}
</script>

<style lang="less" scoped>

</style>
