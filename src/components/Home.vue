<template>
  <div id="home">
    <el-container class="home-container">
      <!-- 头部 -->
      <el-header>
        <div>
          <img
            src="../assets/heima.png"
            alt=""
          >
          <span class="title">电商后台管理系统</span>
        </div>
        <el-button
          type="info"
          @click="logout"
        >退 出</el-button>
      </el-header>
      <!-- 主体 -->
      <el-container>
        <!-- 侧边栏 -->
        <el-aside :width="isCollapse ? '64px' : '200px'">
          <div
            class="toggle-button"
            @click="toggleCollapse"
          >|||</div>
          <!-- 侧边栏菜单区域 -->
          <el-menu
            background-color="#B9E0D3"
            text-color="#2C3E50"
            active-text-color="#315DD3"
            unique-opened
            :collapse="isCollapse"
            :collapse-transition="false"
            :router="true"
            :default-active="activePath"
          >
            <!-- 一级菜单 -->
            <el-submenu
              :index="item.id+''"
              v-for="(item,index) in menulist"
              :key="item.id"
            >
              <!-- 一级菜单模板区域 -->
              <template slot="title">
                <!-- 图标 -->
                <i :class="iconsObj[item.id]"></i>
                <!-- 文本 -->
                <span>{{item.authName}}</span>
              </template>
              <!-- 二级菜单 -->
              <el-menu-item
                :index="'/'+subItem.path"
                v-for="subItem in item.children"
                :key="subItem.id"
                @click="saveNavState('/'+subItem.path)"
              >
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
        <!-- 内容主体 -->
        <el-main>
          <!-- welcome路由占位符 -->
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>

  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      menulist: [],
      iconsObj: {
        125: 'el-icon-s-custom iconF',
        103: 'el-icon-s-finance iconF',
        101: 'el-icon-s-goods iconF',
        102: 'el-icon-s-order iconF',
        145: 'el-icon-s-platform iconF'
      },
      isCollapse: false,
      //被激活的链接地址
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
      this.$message.warning('您已退出，请重新登录')
    },
    //获取所有的菜单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res);
    },
    //菜单的折叠
    toggleCollapse() {
      this.isCollapse = !this.isCollapse;
    },
    //保存链接的点击激活
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      console.log(activePath);
      this.activePath = activePath
    }
  },

}
</script>

<style lang="less">
#home {
  height: 100%;
}
.home-container {
  height: 100%;
}
.el-header {
  background-color: #8bc9b4;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;

  .el-button {
    background-color: #e4f5ef;
    color: #2c3e50;
    border-color: transparent;
    padding: 8px 14px;
    width: 65px;
    height: 35px;
  }
  .el-button:hover {
    background-color: #ebf8f3;
  }
  .title {
    color: #2c3e50;
    font-size: 20px;
    letter-spacing: 2px;
  }

  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #b9e0d3;
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #F2F2F2;
}
.toggle-button {
  background-color: #def0ea;
  font-size: 12px;
  line-height: 24px;
  text-align: center;
  letter-spacing: 1.5px;
  cursor: pointer;
  color: black;
}
</style>
