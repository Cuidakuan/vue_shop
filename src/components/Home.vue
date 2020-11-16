<template>

 <el-container class="home-container">
    <!-- 头部区域  -->
    <el-header>
        <div>
            <img class="head-logo" src="../assets/img1.png" alt="">
            <span> 崔大宽的电商后台管理系统</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
    </el-header>
   <!-- 页面主体区域 -->
    <el-container>
        <!-- 侧边栏 -->
        <el-aside :width="isCollapse?'64px':'200px'">
            <div class="toggle-button" @click="toggleCollapse" >
                |||
            </div>
            <!-- 侧边栏菜单区域 -->
            <el-menu
                :router="true"
                :collapse='isCollapse'
                :collapse-transition=false
                :unique-opened="true"
                background-color="#3B404B"
                text-color="#fff"
                active-text-color="#4293EE"
                :default-active="activePath"
                >
                <!-- 一级菜单 -->
                <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
                    <!-- 一级菜单的模板区域 -->
                    <template slot="title">
                        <!-- 图标 -->
                    <i :class="iconsObj[item.id]"></i>
                    
                    <!-- 文本 -->
                    <span>{{item.authName}}</span>
                    </template>
                    
                    <!-- 二级菜单 -->
                    <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/'+subItem.path)">
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
        <!-- 右侧主体区域 -->
        <el-main>
            <!-- 放一个路由占位符 -->
            <router-view></router-view>
        </el-main>
    </el-container>
    </el-container>
</template>

<script>
export default {
    data(){
        return{
            //左侧菜单数据
            menulist:[],
            iconsObj:{
                125:'el-icon-s-custom',
                103:'el-icon-key',
                101:'el-icon-shopping-bag-1',
                102:'el-icon-tickets',
                145:'el-icon-data-line'
            },
            //是否折叠
            isCollapse:false,
            //被激活的链接地址
            activePath:''
                
            
        }
    },
    created(){
        this.getMenuList(),
        this.activePath=window.sessionStorage.getItem('activePath')
        
    },
    methods:{
        logout(){
            window.sessionStorage.clear()
            this.$router.push('/login')
        },
        //获取所有的菜单
       async getMenuList(){
          const{data:res}= await this.$http.get('menus')
          if(res.meta.status!==200) return this.$message.error(res.meta.msg)
          this.menulist=res.data
          //console.log(res)
        },
        //实现菜单的折叠和展开
        toggleCollapse(){
            this.isCollapse=!this.isCollapse

        },
        //保存链接的激活状态
        saveNavState(activePath){
            window.sessionStorage.setItem('activePath',activePath)
            this.activePath=activePath
        }
    }
}
</script>

<style lang="less" scoped>
.home-container{
    height: 100%;
}

.head-logo{
    height: 60px;
    width: 60px;
    margin-left: 10px;
}
.el-header{
    background-color: #363C3E;
    display:flex ;
    justify-content:space-between ;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    >div{
        display: flex;
        align-items: center;
        span{
            margin-left: 15px;
        }
    }
}
.el-aside{
    background-color: #3C414C;
    .el-menu{
    border-right:none;
}
}
.el-main{
    background-color: #D7DADD;
}
.toggle-button{
    background-color: #4E5668;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing:0.2em ;
    cursor: pointer;
}
</style>