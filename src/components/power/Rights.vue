<template>
  <div class="wrapper">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-table
        :data="rightsList"
        style="width: 100%"
        border
        stripe>
        <el-table-column
          type="index">
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
            <template slot-scope="scope">
              <el-tag v-if="scope.row.level === '0'">一级</el-tag>
              <el-tag type="success" v-if="scope.row.level === '1'">二级</el-tag>
              <el-tag type="warning" v-if="scope.row.level === '2'">三级</el-tag>
            </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      rightsList: []
    }
  },
  created() {
    this.getRightsList()
  },
  mounted() {

  },
  methods: {
    getRightsList(){
      this.$http.get('/rights/list').then(res => {
        const data = res.data
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.rightsList = data.data
      })
    }
  }
}
</script>

<style lang="less" scoped>
.wrapper{
}
</style>
}
