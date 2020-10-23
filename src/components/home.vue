<template>
  <el-container class="home-container">
    <el-header>
      <div>
        <img src="../assets/shi.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout"> 退出 </el-button></el-header>
    <el-container>
      <!-- 侧边栏,宽度根据是否折叠进行设置 -->
      <el-aside :width="isCollapse ? '64px':'200px'">
        <!-- 伸缩侧边栏按钮 -->
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          unique-opened
          router
          :default-active="activePath"
          :collapse="isCollapse"
          :collapse-transition="false"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#8CDCDA">

          <el-submenu :index="item.id+''" v-for="item in this.menusList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
          <el-menu-item @click="saveActivePath(subItem.path)" :index="subItem.path" v-for="subItem in item.children" :key="subItem.id">
            <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>
            </template>
          </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view />
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menusList: '',
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },

  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },

  methods: {
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menusList = res.data
    },

    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },

    saveActivePath (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    },

    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header{
  background-color:#373D41;
  display: flex;
  justify-content: space-between;
  color: #fff;
  align-items: center;
  font-size: 20px;
    div{
      display: flex;
      align-items: center;
      span{
        margin-left: 15px;
      }
    }
  img{
    height: 50px;
    width: 50px;
  }
}
.el-aside{
  background-color:#333744;
}
.el-main{
  background-color:#eaedf1;
}
.toggle-button{
  background-color: cornflowerblue;
  letter-spacing: .2em;
  text-align: center;
  height: 30px;
  line-height: 30px;
  cursor: pointer;
}
</style>
