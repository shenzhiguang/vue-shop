/* eslint-disable quote-props */
<template>
    <el-container class="home_container">
        <!-- 头部区域 -->
        <el-header>
            <div>
                <img src="../assets/heima.png" alt="">
                <span>电商后台管理系统</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <el-container>
            <!-- 侧边栏区域 -->
            <el-aside :width="isCollapse ? '64px' : '200px'">
                <!-- 侧边栏导航 -->
                <div class="toggle_button" @click="toggleCollapse">|||</div>
                <el-menu
                    background-color="#333744"
                    text-color="#fff"
                    active-text-color="#409eff"
                    unique-opened
                    :collapse="isCollapse"
                    :collapse-transition="false"
                    router
                    :default-active="actionActive">
                    <!-- 一级导航 -->
                    <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
                        <!-- 模板区域 -->
                        <template slot="title">
                            <i :class="iconObj[item.id]"></i>
                            <span>{{ item.authName }}</span>
                        </template>
                        <!-- 二级导航 -->
                        <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveActionActive('/' + subItem.path)">
                            <template slot="title">
                                <i class="el-icon-menu"></i>
                                <span>{{ subItem.authName }}</span>
                        </template>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>
            <!-- 主体区域 -->
            <el-main>
                <!-- 路由占位符 -->
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconObj: {
        // eslint-disable-next-line quote-props
        125: 'iconfont icon-users',
        // eslint-disable-next-line quote-props
        '103': 'iconfont icon-tijikongjian',
        // eslint-disable-next-line quote-props
        '101': 'iconfont icon-shangpin',
        // eslint-disable-next-line quote-props
        '102': 'iconfont icon-danju',
        // eslint-disable-next-line quote-props
        '145': 'iconfont icon-baobiao'
      },
      // 菜单是否折叠
      isCollapse: false,
      // 菜单的哪个列表被激活，保存的是被激活的菜单列表项
      actionActive: ''
    }
  },
  created () {
    this.getMenuList()
    // 一刷新，data 中用于保存被激活的菜单列表项的数据就没了，这时就需要通过 sessionStorage
    // 来获取并设置赋值到 data 中
    this.actionActive = window.sessionStorage.getItem('actionActive')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取菜单数据
    async getMenuList () {
      const { data: res } = await this.$http.get('/menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    // 点击按钮切换菜单的折叠与展开
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    // 保存菜单的激活状态
    saveActionActive (actionActive) {
      window.sessionStorage.setItem('actionActive', actionActive)
      this.actionActive = actionActive
    }
  }
}
</script>

<style lang="less" scoped>
.home_container {
    height: 100%;
}

.el-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #373d41;
    padding-left: 10px;
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
}

.el-main {
    background-color: #eaedf1;
}

.iconfont {
    margin-right: 8px;
}

.el-menu {
    border-right: 0;
}

.toggle_button {
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    background-color: #4a5064;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>
