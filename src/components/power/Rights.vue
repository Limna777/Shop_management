<template>
  <div>
    <!-- 导航面包屑 -->
    <el-breadcrumb>
      <el-breadcrumb-item :to="{ path: '/rights' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片列表 -->
    <el-card class="box-card">
      <template>
        <el-table
          stripe
          :data="rightsList"
          border
          style="width: 100%"
        >
          <el-table-column type="index">
          </el-table-column>
          <el-table-column
            prop="authName"
            label="权限名称"
          >
          </el-table-column>
          <el-table-column
            prop="path"
            label="路径"
          >
          </el-table-column>
          <el-table-column
            prop="level"
            label="权限等级"
          >
            <template slot-scope="scope">
              <el-tag v-if="scope.row.level == '0'" >一级</el-tag>
              <el-tag type="success" v-else-if="scope.row.level ==='1'">二级</el-tag>
              <el-tag type="warning" v-else>三级</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </template>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Rights',
  data() {
    return {
      rightsList: []
    }
  },
  created() {
    this.getRightList()
  },
  methods: {
    async getRightList() {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      console.log(res);
      this.rightsList = res.data
    }
  }
}
</script>

<style scoped>
</style>
