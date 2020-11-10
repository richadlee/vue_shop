<template>
  <div class="wrapper">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table :data="catelist"
      :columns="columns"
      :selection-type="false"
      :expand-type="false"
      show-index
      index-text="#"
      border
      :show-row-hover="false"
      class="treeTable">
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="!scope.row.cat_deleted" style="color: lightgreen;"></i>
          <i class="el-icon-error" v-else style="color: red;"></i>
        </template>
        <!--排序  -->
        <template slot="order" slot-scope="scope">
          <el-tag type="primary" size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="openEditCateDialog(scope.row)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="delCate(scope.row)">删除</el-button>
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
      <!-- 添加分类对话框 -->
      <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="50%"
        @close="addCateDialogClosed">
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRules" label-width="100px">
          <el-form-item label="分类名称: " prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类: ">
            <!-- 指定数据源 -->
            <el-cascader
              clearable
              style="width: 100%;"
              v-model="selectedKeys"
              :options="parenCatelist"
              :props="cascaderProps"
              @change="parentCateChanged"></el-cascader>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑分类对话框 -->
      <el-dialog
        title="修改分类"
        :visible.sync="editCateDialogVisible"
        width="50%"
        @close="editCateDialogClosed">
        <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px">
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="editCateForm.cat_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editCateName">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表
      catelist: [],
      // 总数据条数
      total: 0,
      // 为table指定列
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'isok'
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
      ],
      // 控制添加分类的对话框显示
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加的分类名称
        cat_name: '',
        // 父级分类ID
        cat_pid: 0,
        // 分类层级
        cat_level: 0
      },
      // 添加分类的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类列表
      parenCatelist: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 选中的父级分类ID数组
      selectedKeys: [],
      // 编辑对话框显示隐藏
      editCateDialogVisible: false,
      editCateForm: {
        cat_name: ''
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
        ]
      },
      editCateId: ''
    }
  },
  created() {
    this.getCateList()
  },
  mounted() {

  },
  methods: {
    // 获取商品分类数据
    getCateList(){
      this.$http.get('categories', { params: this.queryInfo }).then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.catelist = data.data.result
        this.total = data.data.total
      })
    },
    // 监听pagesize改变
    handleSizeChange(newSize){
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听当前页码改变
    handleCurrentChange(newPage){
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮展示添加分类对话框
    showAddCateDialog(){
      // 获取父级分类的列表数据
      this.getParentCatelist()
      this.addCateDialogVisible = true
    },
    // 获取父级分类数据列表
    getParentCatelist(){
      this.$http.get('categories', { params: { type: 2 } }).then(res => {
        const { data } = res
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.parenCatelist = data.data
      })
    },
    // 选择项发生变化触发
    parentCateChanged(){
      if (this.selectedKeys.length > 0){
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      }
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 点击按钮添加新的分类
    addCate(){
      this.$refs.addCateFormRules.validate(valid => {
        if (!valid) return
        this.$http.post('categories', this.addCateForm).then(res => {
          const { data } = res
          if (data.meta.status !== 201) return this.$message.error(data.meta.msg)
          this.$message.success(data.meta.msg)
          this.getCateList()
        })
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框关闭重置数据
    addCateDialogClosed(){
      this.$refs.addCateFormRules.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.pid = 0
      this.addCateForm.cat_name = ''
    },
    // 监听修改分类名称对话框关闭重置数据
    editCateDialogClosed(){
      this.$refs.editCateFormRef.resetFields()
      this.editCateId = ''
      this.editCateForm.cat_name = ''
    },
    // 打开修改分类名称对话框
    openEditCateDialog(data){
      this.editCateId = data.cat_id
      this.editCateForm.cat_name = data.cat_name
      this.editCateDialogVisible = true
    },
    // 提交修改的分类名称
    editCateName(){
      this.$refs.editCateFormRef.validate(valid => {
        if (!valid) return
        this.$http.put(`categories/${this.editCateId}`, this.editCateForm).then(res => {
          const { data } = res
          if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
          this.$message.success(data.meta.msg)
          this.getCateList()
          this.editCateDialogVisible = false
        })
      })
    },
    // 删除分类
    delCate(data){
      this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete(`categories/${data.cat_id}`).then(res => {
          const { data } = res
          if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
          this.$message.success(data.meta.msg)
          this.getCateList()
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
.wrapper{

}
.treeTable{
  margin-top: 15px;
}
</style>
}
