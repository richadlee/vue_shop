<template>
  <div class="wrapper">
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片式图 -->
    <el-card >
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="9">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表 -->
      <el-table
        border
        stripe
        :data="userList"
        style="width: 100%">
        <el-table-column
          type="index"
          label="#">
        </el-table-column>
        <el-table-column
          prop="username"
          label="姓名">
        </el-table-column>
        <el-table-column
          prop="email"
          label="邮箱">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="电话">
        </el-table-column>
        <el-table-column
          prop="role_name"
          label="角色">
        </el-table-column>
        <el-table-column
          label="状态">
            <template slot-scope="scope">
              <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)"></el-switch>
            </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="180px">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
            <!-- 分配角色 -->
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button @click="setRole(scope.row)" type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 内容主体 -->
    <el-form ref="addFormRef" :model="addForm" label-width="70px" :rules="addFormRules">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="addForm.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password" >
        <el-input v-model="addForm.password" type="password"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="addForm.email"></el-input>
      </el-form-item>
      <el-form-item label="手机" prop="mobile">
        <el-input v-model.number="addForm.mobile"></el-input>
      </el-form-item>
    </el-form>
      <!-- 底部 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
    <el-form ref="editFormRef" :model="editForm" label-width="80px" :rules="editFormRules">
      <el-form-item label="用户">
        <el-input v-model="editForm.username" disabled></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="editForm.email" ></el-input>
      </el-form-item>
      <el-form-item label="手机" prop="mobile">
        <el-input v-model.number="editForm.mobile" ></el-input>
      </el-form-item>
    </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色对话框 -->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoleDialogVisible"
      width="50%"
      @close="setRoleDialogClosed">
      <div>
        <p>当前用户: {{userInfo.username}} </p>
        <p>当前角色: {{userInfo.role_name}} </p>
        <p>分配角色:
          <el-select v-model="selectedRoleId" placeholder="请选择">
            <el-option
              v-for="item in rolesList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id">
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    const checkMobile = (rule, value, cb) => {
      const reg = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (reg.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号码'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      userList: [],
      total: 0,
      // 控制添加用户对话框的显示和隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名长度在3~10字符之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '密码长度在6~15字符之间', trigger: 'blur' }
        ],
        email: [
          { type: 'email', required: true, message: '请输入正确的邮箱地址', trigger: 'blur' }
        ],
        mobile: [
          { type: 'number', required: true, message: '请输入正确的手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ],
      },
      // 控制修改用户对话框的显示隐藏
      editDialogVisible: false,
      // 修改对话框中的查询到的用户信息
      editForm: {},
      editFormRules: {
        email: [
          { type: 'email', required: true, message: '请输入邮箱', trigger: 'blur' },
        ],
        mobile: [
          { type: 'number', required: true, message: '请输入正确的手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 分配角色对话框的显示与隐藏
      setRoleDialogVisible: false,
      // 角色对话框的用户信息
      userInfo: {},
      // 角色列表
      rolesList: [],
      // 被选中的新分配角色的ID
      selectedRoleId: ''
    }
  },
  created() {
    this.getUserList()
  },
  mounted() {
  },
  methods: {
    getUserList(){
      this.$http.get('users', { params: this.queryInfo }).then(res => {
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.userList = res.data.data.users
        this.total = res.data.data.total
      })
    },
    // 监听pagesize改变
    handleSizeChange(newSize){
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码改变
    handleCurrentChange(newPage){
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听用户状态修改
    userStateChange(userInfo){
      this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`).then(res => {
        if (res.data.meta.status !== 200) {
          userInfo.mg_state = !userInfo.mg_state
          return this.$message.error(res.data.meta.msg)
        }
        this.$message.success(res.data.meta.msg)
      })
    },
    // 关闭对话框事件处理
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    // 添加用户
    addUser(){
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return
        this.$http.post('users', this.addForm).then(res => {
          const data = res.data
          if (data.meta.status !== 201) return this.$message.error(data.meta.msg)
          this.addDialogVisible = false
          this.$message.success(data.meta.msg)
          this.getUserList()
        })
      })
    },
    // 编辑按钮事件处理
    async showEditDialog(id){
      await this.$http.get('users/' + id).then(res => {
        const data = res.data
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.editForm = data.data
      })
      this.editDialogVisible = true
    },
    // 监听修改用户对话框关闭事件
    editDialogClosed(){
      this.$refs.editFormRef.resetFields()
    },
    editUser(){
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return
        this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile }).then(res => {
          const data = res.data
          if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
          this.editDialogVisible = false
          this.$message.success(data.meta.msg)
          this.getUserList()
        })
      })
    },
    // 通过ID删除用户
    removeUserById(id){
      this.$confirm('此操作将永久删除该用户,是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete('users/' + id).then(res => {
          const data = res.data
          if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
          this.$message.success(data.meta.msg)
          this.getUserList()
        })
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    // 展示分配角色的对话框
    setRole(userInfo){
      this.userInfo = userInfo
      // 获取角色列表
      this.$http.get('roles').then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.rolesList = data.data
      })
      this.setRoleDialogVisible = true
    },
    // 点击按钮分配角色
    saveRoleInfo(){
      if (!this.selectedRoleId) return this.$message.error('请选择要分配的角色')
      this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId }).then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.$message.success(data.meta.msg)
        this.getUserList()
        this.setRoleDialogVisible = false
      })
    },
    // 监听分配角色对话框关闭的事件处理
    setRoleDialogClosed(){
      this.selectedRoleId = ''
    }
  }
}
</script>

<style lang="less" scoped>
.wrapper{
}
</style>
