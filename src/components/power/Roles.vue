<template>
  <div class="wrapper">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id" :class="['bd-bottom', i1 === 0 ? 'bd-top' : '']">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag @close="removeRightById(scope.row,item1.id)" closable>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过FOR循环嵌套渲染二级权限 -->
                <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2 === 0 ? '' : 'bd-top']">
                  <el-col :span="6">
                    <el-tag closable @close="removeRightById(scope.row,item2.id)" type="success">{{item2.authName}} </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag closable @close="removeRightById(scope.row,item3.id)" type="warning" v-for="(item3) in item2.children" :key=" item3.id">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色" prop="roleName" width="250px"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc" width="300px"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRolesById(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 内容主体 -->
    <el-form ref="addFormRef" :model="addForm" label-width="100px" :rules="addFormRules">
      <el-form-item label="角色名称" prop="roleName">
        <el-input v-model="addForm.roleName"></el-input>
      </el-form-item>
      <el-form-item label="角色描述" prop="roleDesc" >
        <el-input v-model="addForm.roleDesc"></el-input>
      </el-form-item>
    </el-form>
      <!-- 底部 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
    <el-form ref="editFormRef" :model="editForm" label-width="100px" :rules="editFormRules">
      <el-form-item label="角色 ID">
        <el-input v-model="editForm.roleId" disabled></el-input>
      </el-form-item>
      <el-form-item label="角色名称" prop="roleName">
        <el-input v-model="editForm.roleName" ></el-input>
      </el-form-item>
      <el-form-item label="角色描述" prop="roleDesc">
        <el-input v-model.number="editForm.roleDesc" ></el-input>
      </el-form-item>
    </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree ref="treeRef" :default-checked-keys="defKeys" default-expand-all node-key="id" show-checkbox :data="rightslist" :props="treeProps" ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      // 所有角色列表数据
      rolesList: [],
      // 增加角色表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 增加角色表单验证规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 1, max: 10, message: '角色长度在1~10字符之间', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
        ]
      },
      // 控制增加角色对话框的显示和隐藏
      addDialogVisible: false,
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 1, max: 10, message: '角色长度在1~10字符之间', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
        ]
      },
      // 控制分配权限对话框
      setRightDialogVisible: false,
      // 所有权限数据
      rightslist: [],
      // 树形空间绑定对象
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的节点ID值
      defKeys: [],
      // 当前即将分配角色权限的角色ID
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  mounted() {

  },
  methods: {
    // 获取角色列表
    getRolesList(){
      this.$http.get('roles').then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.rolesList = data.data
      })
    },
    // 关闭添加角色对话框事件处理
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    // 添加角色
    addRoles(){
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return
        this.$http.post('roles', this.addForm).then(res => {
          const { data } = res
          if (data.meta.status !== 201) return this.$message.error(data.meta.msg)
          this.addDialogVisible = false
          this.$message.success(data.meta.msg)
          this.getRolesList()
        })
      })
    },
    // 通过ID删除角色
    removeRolesById(id){
      this.$confirm('此操作将永久删除该角色,是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete('roles/' + id).then(res => {
          const data = res.data
          if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
          this.$message.success(data.meta.msg)
          this.getRolesList()
        })
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    // 编辑按钮事件处理
    async showEditDialog(id){
      await this.$http.get('roles/' + id).then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.editForm = data.data
      })
      this.editDialogVisible = true
    },
    // 监听修改用户对话框关闭事件
    editDialogClosed(){
      this.$refs.editFormRef.resetFields()
    },
    // 编辑用户
    editUser(){
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return
        this.$http.put('roles/' + this.editForm.id, this.editForm).then(res => {
          const data = res.data
          if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
          this.editDialogVisible = false
          this.$message.success(data.meta.msg)
          this.getRolesList()
        })
      })
    },
    // 根据ID删除权限
    removeRightById(role, rightId){
      this.$confirm('此操作将删除该角色的权限,是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete('roles/' + role.id + '/rights/' + rightId).then(res => {
          const data = res.data
          if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
          this.$message.success(data.meta.msg)
          role.children = data.data
        })
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    // 展示分配权限对话框
    showSetRightDialog(role){
      this.roleId = role.id
      // 获取所有权限的数据
      this.$http.get('rights/tree').then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.rightslist = data.data
        console.log(this.rightslist)
        console.log(role)
        this.getLeafKeys(role, this.defKeys)
        this.setRightDialogVisible = true
      })
    },
    // 通过递归方法获取三级权限ID数组
    getLeafKeys(node, arr){
      // 如果当前node节点不包含children属性,则是三级节点返回ID
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed(){
      this.defKeys = []
    },
    // 点击为角色分配权限
    allotRights(){
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr }).then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.$message.success(data.meta.msg)
        this.getRolesList()
        this.setRightDialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
.wrapper{
}
.el-tag{
  margin: 7px;
}
.bd-top {
  border-top: 1px solid #eee;
}
.bd-bottom {
  border-bottom: 1px solid #eee;
}
.el-row {
  display: flex;
  align-items: center;
}
</style>
