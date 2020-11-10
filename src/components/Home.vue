<template>
    <el-container class="home-container">
      <!-- Header -->
      <el-header>
        <div>
          <img src="../assets/heima.png" alt="logo">
          <span>电商管理后台</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
      </el-header>
      <!-- MainPage -->
      <el-container>
        <!-- Aside -->
        <el-aside width="unset">
          <div class="toggle-button" @click="toggleCollapse">|||</div>
          <!-- Aside Menu -->
          <el-menu
            router
            class="el-menu-vertical-demo"
            :default-active="activePath"
            :collapse="isCollapse"
            unique-opened
            background-color="#333744"
            text-color="#fff"
            active-text-color="#409EFF">
            <!-- 一级菜单 -->
            <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
              <!-- 一级菜单模板区域 -->
              <template slot="title">
                <!-- 图标 -->
                <i :class="iconsObj[item.id]"></i>
                <!-- 文本 -->
                <span>{{item.authName}}</span>
              </template>
                <!-- 二级菜单 -->
                <el-menu-item :index="subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState(subItem.path)">
                  <template slot="title">
                    <!-- 图标 -->
                    <i class="el-icon-menu"></i>
                    <!-- 文本 -->
                    <span>{{subItem.authName}}</span>
                  </template>
                </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>
        <!-- Content -->
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      // 左侧菜单
      menulist: [],
      iconsObj: {
        125: 'el-icon-user-solid',
        103: 'el-icon-s-operation',
        101: 'el-icon-s-grid',
        102: 'el-icon-document-copy',
        145: 'el-icon-s-data'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  mounted() {},
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('./login')
    },
    // 获取所有菜单
    getMenuList() {
      this.$http.get('menus').then(res => {
        const data = res.data
        if (data.meta.status !== 200) return this.$message.error(data.meta.msg)
        this.menulist = data.data
      })
    },
    // 点击按钮切换菜单折叠
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    // 保存链接的激活状态
    saveNavState(path){
      window.sessionStorage.setItem('activePath', path)
      this.activePath = path
    }
  },
}
</script>

<style lang="less" scoped>
.el-header {
  background-color: #373D41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
      span {
        margin-left: 15px;
      }
  }
}

.el-aside {
  background-color: #333744;
  .el-menu {
    border-right: none;
  }
}

.el-main {
  background-color: #eaedf1;
}

.home-container {
  height: 100%;
}

.toggle-button {
  background-color: #4A5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer
}

.el-menu-vertical-demo:not(.el-menu--collapse) {
  width: 200px;
  min-height: 400px;
}
</style>
