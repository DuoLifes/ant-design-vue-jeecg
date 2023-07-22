<template>
  <!-- , width: fixedHeader ? `calc(100% - ${sidebarOpened ? 256 : 80}px)` : '100%'  -->
  <a-layout-header
    v-if="!headerBarFixed"
    :class="[fixedHeader && 'ant-header-fixedHeader', sidebarOpened ? 'ant-header-side-opened' : 'ant-header-side-closed', ]"
    :style="{ padding: '0' }">

    <div v-if="mode === 'sidemenu'" :class="['top-nav-header-index', theme]">
      <a-icon
        v-if="device==='mobile'"
        class="trigger"
        :type="collapsed ? 'menu-fold' : 'menu-unfold'"
        @click="toggle"></a-icon>
      <a-icon
        v-else
        class="trigger"
        :type="collapsed ? 'menu-unfold' : 'menu-fold'"
        @click="toggle"/>

        <a-menu  mode="horizontal" :theme="theme" v-model:selectedKeys="current" @select="selectHeader" class="top-menu-dark">
          <a-menu-item  v-for="m in menus" :key="m.id">
            {{ m.meta.title }}
          </a-menu-item>
        </a-menu>

      <!-- <span v-if="device === 'desktop'">欢迎进入 Jeecg-Boot 企业级低代码平台</span>
      <span v-else>Jeecg-Boot</span> -->

      <user-menu :theme="theme"/>
    </div>
    <!-- 顶部导航栏模式 -->
    <div v-else :class="['top-nav-header-index', theme]">
      <div class="header-index-wide">
        <div class="header-index-left" :style="topMenuStyle.headerIndexLeft">
          <logo class="top-nav-header" :show-title="device !== 'mobile'" :style="topMenuStyle.topNavHeader"/>
          <div v-if="device !== 'mobile'" :style="topMenuStyle.topSmenuStyle">
            <s-menu
              mode="horizontal"
              :menu="menus"
              :theme="theme"
              @updateMenuTitle="handleUpdateMenuTitle"
            ></s-menu>
          </div>
          <a-icon
            v-else
            class="trigger"
            :type="collapsed ? 'menu-fold' : 'menu-unfold'"
            @click="toggle"></a-icon>
        </div>
        <user-menu class="header-index-right" :theme="theme" :style="topMenuStyle.headerIndexRight"/>
      </div>
    </div>

  </a-layout-header>
</template>

<script>
  import UserMenu from '../tools/UserMenu'
  import SMenu from '../menu/'
  import Logo from '../tools/Logo'
  import { mixin } from '@/utils/mixin.js'

  export default {
    name: 'GlobalHeader',
    components: {
      UserMenu,
      SMenu,
      Logo,
    },
    mixins: [mixin],
    props: {
      mode: {
        type: String,
        // sidemenu, topmenu
        default: 'sidemenu'
      },
      menus: {
        type: Array,
        required: true
      },
      theme: {
        type: String,
        required: false,
        default: 'dark'
      },
      collapsed: {
        type: Boolean,
        required: false,
        default: false
      },
      device: {
        type: String,
        required: false,
        default: 'desktop'
      }
    },
    data() {
      return {
        headerBarFixed: false,
        //update-begin--author:sunjianlei---date:20190508------for: 顶部导航栏过长时显示更多按钮-----
        topMenuStyle: {
          headerIndexLeft: {},
          topNavHeader: {},
          headerIndexRight: {},
          topSmenuStyle: {}
        },
        chatStatus: '',
        current:[],
        cacheMenu:[]
      }
    },
    watch: {
      /** 监听设备变化 */
      device() {
        if (this.mode === 'topmenu') {
          this.buildTopMenuStyle()
        }
      },
      /** 监听导航栏模式变化 */
      mode(newVal) {
        if (newVal === 'topmenu') {
          this.buildTopMenuStyle()
        }
      }
    },

    //update-end--author:sunjianlei---date:20190508------for: 顶部导航栏过长时显示更多按钮-----
    mounted() {
      // 给所有子菜单添加parentId
      this.getParentUrl()
      // 根据url选中侧栏和顶栏菜单
      this.getRouter(this.cacheMenu,this.$route.path)
      window.addEventListener('scroll', this.handleScroll)
      //update-begin--author:sunjianlei---date:20190508------for: 顶部导航栏过长时显示更多按钮-----
      if (this.mode === 'topmenu') {
        this.buildTopMenuStyle()
      }
      this.$eventBus.$on('change_tabs', (key)=>{
        this.getRouter(this.cacheMenu,key)
      })
      //update-end--author:sunjianlei---date:20190508------for: 顶部导航栏过长时显示更多按钮-----
    },
    methods: {
      getRouter(list,path){
        list.forEach(element => {
          if(path === element.path){
            this.current = [element.parentId]
            this.$emit('updateLeftMenu',{key:element.parentId},true)
          } else {
            if(element.children){
              this.getRouter(element.children,path)
            }
          }
        });
      },
      getParentUrl(){
        this.menus.forEach(element => {
          let obj = JSON.parse(JSON.stringify(element))
          obj.parentId = obj.id
          if(obj.children){
            this.getTreeNode(obj)
          }
          this.cacheMenu.push(obj)
        })
      },
      getTreeNode(obj){
        obj.children.forEach(element => {
          element.parentId = obj.parentId
          if(element.children){
            this.getTreeNode(element)
          }
        })
      },
      selectHeader(item){
        this.$emit('updateLeftMenu',item,false)
      },
      handleScroll() {
        if (this.autoHideHeader) {
          let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
          if (scrollTop > 100) {
            this.headerBarFixed = true
          } else {
            this.headerBarFixed = false
          }
        } else {
          this.headerBarFixed = false
        }
      },
      toggle() {
        this.$emit('toggle')
      },
      //update-begin--author:sunjianlei---date:20190508------for: 顶部导航栏过长时显示更多按钮-----
      buildTopMenuStyle() {
        if (this.mode === 'topmenu') {
          if (this.device === 'mobile') {
            // 手机端需要清空样式，否则显示会错乱
            this.topMenuStyle.topNavHeader = {}
            this.topMenuStyle.topSmenuStyle = {}
            this.topMenuStyle.headerIndexRight = {}
            this.topMenuStyle.headerIndexLeft = {}
          } else {
            let rightWidth = '400px'
            this.topMenuStyle.topNavHeader = { 'min-width': '165px' }
            this.topMenuStyle.topSmenuStyle = { 'width': 'calc(100% - 165px)' }
            this.topMenuStyle.headerIndexRight = { 'min-width': rightWidth, 'white-space': 'nowrap' }
            this.topMenuStyle.headerIndexLeft = { 'width': `calc(100% - ${rightWidth})` }
          }
        }
      },
      //update-begin--author:sunjianlei---date:20190508------for: 顶部导航栏过长时显示更多按钮-----

      // update-begin-author:sunjianlei date:20210508 for: 修复动态功能测试菜单、带参数菜单标题错误、展开错误的问题
      handleUpdateMenuTitle(value) {
        this.$emit('updateMenuTitle', value)
      },
      // update-end-author:sunjianlei date:20210508 for: 修复动态功能测试菜单、带参数菜单标题错误、展开错误的问题

    }
  }
</script>

<style lang="less" scoped>
  /* update_begin author:scott date:20190220 for: 缩小首页布局顶部的高度*/

  @height: 59px;

  .layout {

    .top-nav-header-index {

      .header-index-wide {
        margin-left: 10px;

        .ant-menu.ant-menu-horizontal {
          height: @height;
          line-height: @height;
        }
      }
      .trigger {
        line-height: 64px;
        color: #fff;
        &:hover {
          background: rgba(0, 0, 0, 0.05);
        }
      }
    }

    .header {
      z-index: 2;
      color: white;
      height: @height;
      background-color: @primary-color;
      transition: background 300ms;

      /* dark 样式 */
      &.dark {
        color: #000000;
        box-shadow: 0 0 4px rgba(0, 0, 0, 0.2);
        background-color: white !important;
      }
    }

    .header, .top-nav-header-index {
      &.dark .trigger:hover {
        background: rgba(0, 0, 0, 0.05);
      }
    }
  }

  .ant-layout-header {
    height: @height;
    line-height: @height;
  }
  .top-menu-dark{
    display: inline-block;
    height: @height;
    position: absolute;
    line-height: @height;
  }

  /* update_end author:scott date:20190220 for: 缩小首页布局顶部的高度*/

</style>