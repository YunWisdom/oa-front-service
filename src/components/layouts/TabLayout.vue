<template>
  <global-layout @dynamicRouterShow="dynamicRouterShow" @closeTabPage="closeTabPage">
    <contextmenu :itemList="menuItemList" :visible.sync="menuVisible" @select="onMenuSelect" />
    <a-tabs
      @contextmenu.native="e => onContextmenu(e)"
      v-if="multipage"
      :active-key="activePage"
      class="tab-layout-tabs"
      style="height:52px"
      :hide-add="true"
      type="editable-card"
      @change="changePage"
      @edit="editPage"
    >
      <a-tab-pane :id="page.fullPath" :key="page.fullPath" v-for="page in pageList">
        <span slot="tab" :pagekey="page.fullPath">{{ page.meta.title }}</span>
      </a-tab-pane>
    </a-tabs>
    <div style="margin: 12px 12px 0;">
      <transition name="page-toggle">
        <keep-alive v-if="multipage">
          <router-view />
        </keep-alive>
        <router-view v-else />
      </transition>
    </div>
  </global-layout>
</template>

<script>
import GlobalLayout from "@/components/page/GlobalLayout";
import Contextmenu from "@/components/menu/Contextmenu";
import { mixin, mixinDevice } from "@/utils/mixin.js";
import { deNull } from "@/utils/util";
import _ from "underscore";

const indexKey = `/dashboard/analysis`;
const workplaceKey = `/dashboard/workplace`;
const centerKey = `/account/center`;

//Tab标题
const titleKV = {
  "0b511f234f3847baa50106a14fff6215": "审批处理",
  b0ceb7cfb2b0487a96e03f50c413d762: "请假申请",
  "933e21cf445440abb8cfdae366082a62": "外出申请",
  "9ed5bc42eb934bbe8dac16ed1a3b103f": "加班申请",
  b0fee76c2ee84baeb9494b196b779e3e: "考勤异常",
  "3da23c7955d84465a759d0f1830a0d00": "档案及证照借阅",
  cfd4ee2d60fa48f585c157d524b0a108: "用印申请(非合同)",
  dd78c4c8a59e4ee3bd93ec011a3f6b4c: "用印申请(合同)",
  e412b58db17b4cbf8cb9833c118c2d3b: "印章借用",
  "62f7122c73c244119e5d4ec8aa170a3d": "出差申请",
  "3972c2cb5a824aaab8ef0aad3ea56e7f": "行政公告",
  c9c7e537dbb54691b4f0eff23854fdbb: "红头文件",
  ebe9412bb4f34b5c8ea3fb7c0908d315: "新闻资讯",
  "6650f6e79479489192fe469d1e3f98f3": "奖罚通报",
  "6bfb3ee6257a4066b1d26f195da83fe0": "车辆补贴"
};

//

export default {
  name: "TabLayout",
  components: {
    GlobalLayout,
    Contextmenu
  },
  mixins: [mixin, mixinDevice],
  data() {
    return {
      pageList: [],
      linkList: [],
      activePage: "",
      menuVisible: false,
      menuItemList: [
        { key: "1", icon: "arrow-left", text: "关闭左侧" },
        { key: "2", icon: "arrow-right", text: "关闭右侧" },
        { key: "3", icon: "close", text: "关闭其它" }
      ]
    };
  },
  computed: {
    multipage() {
      //判断如果是手机模式，自动切换为单页面模式
      if (this.isMobile()) {
        return false;
      } else {
        return this.$store.state.app.multipage;
      }
    }
  },
  created() {
    //是否存在路径标识,检查pageList中是否含有indexKey
    let flag = _.find(this.pageList, item => {
      return item.path == indexKey;
    });
    if (flag == null || typeof flag == "undefined") {
      this.pageList.push({
        name: "dashboard-analysis",
        path: indexKey,
        fullPath: indexKey,
        meta: {
          icon: "dashboard",
          title: "首页"
        }
      });
      this.linkList.push(indexKey);
    }
    //是否存在路径标识,检查pageList中是否含有workplaceKey
    flag = _.find(this.pageList, item => {
      return item.path == centerKey;
    });
    if (flag == null || typeof flag == "undefined") {
      this.pageList.push({
        name: "account-center",
        path: centerKey,
        fullPath: centerKey,
        meta: {
          icon: "account",
          title: "个人中心"
        }
      });
      this.linkList.push(centerKey);
    }
    //是否存在路径标识,检查pageList中是否含有workplaceKey
    flag = _.find(this.pageList, item => {
      return item.path == workplaceKey;
    });
    if (flag == null || typeof flag == "undefined") {
      this.pageList.push({
        name: "dashboard-workplace",
        path: workplaceKey,
        fullPath: workplaceKey,
        meta: {
          icon: "dashboard",
          title: "工作台"
        }
      });
      this.linkList.push(workplaceKey);
    }
    if (
      this.$route.fullPath != indexKey &&
      this.$route.fullPath != workplaceKey &&
      this.$route.fullPath != centerKey
    ) {
      this.pageList.push(this.$route);
      this.linkList.push(this.$route.fullPath);
    }

    this.activePage = this.$route.fullPath;

    //把remove函数，设置到$root上
    this.$root.$tabs = this;
    this.$root.$tabs.closeTab = this.$options.methods.closeTabPage;
  },

  watch: {
    $route: function(newRoute) {
      //设置Tab栏标题
      if (
        typeof titleKV[newRoute.params.code] != "undefined" &&
        titleKV[newRoute.params.code] != "" &&
        titleKV[newRoute.params.code] != null
      ) {
        newRoute.meta.title = titleKV[newRoute.params.code];
      }
      this.activePage = newRoute.fullPath;
      if (!this.multipage) {
        this.linkList = [newRoute.fullPath];
        this.pageList = [Object.assign({}, newRoute)];
      } else if (this.linkList.indexOf(newRoute.fullPath) < 0) {
        this.linkList.push(newRoute.fullPath);
        this.pageList.push(Object.assign({}, newRoute));
      } else if (this.linkList.indexOf(newRoute.fullPath) >= 0) {
        let oldIndex = this.linkList.indexOf(newRoute.fullPath);
        let oldPositionRoute = this.pageList[oldIndex];
        this.pageList.splice(
          oldIndex,
          1,
          Object.assign({}, newRoute, { meta: oldPositionRoute.meta })
        );
      }
    },
    activePage: function(key) {
      //获取激活Index
      var index = this.linkList.lastIndexOf(key);
      //获取激活对象
      var waitRouter = this.pageList[index];
      //获取拷贝对象
      var obj = Object.assign({}, waitRouter);
      //设置动态路由
      try {
        if (deNull(titleKV[obj.params.code]) != "") {
          this.dynamicRouterShow(key, titleKV[obj.params.code]);
        }
      } catch (error) {
        console.log(error);
      }
      //跳转到激活页面
      try {
        this.$router.push(obj.fullPath);
      } catch (error) {
        console.log(error);
      }
    },
    multipage: function(newVal) {
      if (!newVal) {
        this.linkList = [this.$route.fullPath];
        this.pageList = [this.$route];
      }
    },
    $root: () => {}
  },
  methods: {
    changePage(key) {
      this.activePage = key;
    },
    editPage(key, action) {
      this[action](key);
    },
    remove(key) {
      if (key == indexKey) {
        this.$message.warning("首页不能关闭!");
        return;
      }
      if (key == workplaceKey) {
        this.$message.warning("工作台不能关闭!");
        return;
      }
      if (key == centerKey) {
        this.$message.warning("个人中心不能关闭!");
        return;
      }
      if (this.pageList.length === 1) {
        this.$message.warning("这是最后一页，不能再关闭了啦");
        return;
      }
      this.pageList = this.pageList.filter(item => item.fullPath !== key);
      let index = this.linkList.indexOf(key);
      this.linkList = this.linkList.filter(item => item !== key);
      index = index >= this.linkList.length ? this.linkList.length - 1 : index;
      this.activePage = this.linkList[index];
    },
    onContextmenu(e) {
      const pagekey = this.getPageKey(e.target);
      if (pagekey !== null) {
        e.preventDefault();
        this.menuVisible = true;
      }
    },
    getPageKey(target, depth) {
      depth = depth || 0;
      if (depth > 2) {
        return null;
      }
      let pageKey = target.getAttribute("pagekey");
      pageKey =
        pageKey ||
        (target.previousElementSibling
          ? target.previousElementSibling.getAttribute("pagekey")
          : null);
      return (
        pageKey ||
        (target.firstElementChild
          ? this.getPageKey(target.firstElementChild, ++depth)
          : null)
      );
    },
    onMenuSelect(key, target) {
      let pageKey = this.getPageKey(target);
      switch (key) {
        case "1":
          this.closeLeft(pageKey);
          break;
        case "2":
          this.closeRight(pageKey);
          break;
        case "3":
          this.closeOthers(pageKey);
          break;
        default:
          break;
      }
    },
    closeOthers(pageKey) {
      let index = this.linkList.indexOf(pageKey);
      if (pageKey == indexKey) {
        this.linkList = this.linkList.slice(index, index + 1);
        this.pageList = this.pageList.slice(index, index + 1);
        this.activePage = this.linkList[0];
      } else {
        let indexContent = this.pageList.slice(0, 1)[0];
        this.linkList = this.linkList.slice(index, index + 1);
        this.pageList = this.pageList.slice(index, index + 1);
        this.linkList.unshift(indexKey);
        this.pageList.unshift(indexContent);
        this.activePage = this.linkList[1];
      }
    },
    closeLeft(pageKey) {
      if (pageKey == indexKey) {
        return;
      }
      let tempList = [...this.pageList];
      let indexContent = tempList.slice(0, 1)[0];
      let index = this.linkList.indexOf(pageKey);
      this.linkList = this.linkList.slice(index);
      this.pageList = this.pageList.slice(index);
      this.linkList.unshift(indexKey);
      this.pageList.unshift(indexContent);
      if (this.linkList.indexOf(this.activePage) < 0) {
        this.activePage = this.linkList[0];
      }
    },
    closeRight(pageKey) {
      let index = this.linkList.indexOf(pageKey);
      this.linkList = this.linkList.slice(0, index + 1);
      this.pageList = this.pageList.slice(0, index + 1);
      if (this.linkList.indexOf(this.activePage < 0)) {
        this.activePage = this.linkList[this.linkList.length - 1];
      }
    },
    //update-begin-author:taoyan date:20190430 for:动态路由title显示配置的菜单title而不是其对应路由的title
    dynamicRouterShow(key, title) {
      let keyIndex = this.linkList.indexOf(key);
      if (keyIndex >= 0) {
        let currRouter = this.pageList[keyIndex];
        let meta = Object.assign({}, currRouter.meta, { title: title });
        this.pageList.splice(
          keyIndex,
          1,
          Object.assign({}, currRouter, { meta: meta })
        );
      }
    },
    closeTabPage(key) {
      if (key == indexKey) {
        this.$message.warning("首页不能关闭!");
        return;
      }
      if (key == workplaceKey) {
        this.$message.warning("工作台不能关闭!");
        return;
      }
      if (key == centerKey) {
        this.$message.warning("个人中心不能关闭!");
        return;
      }
      if (this.$root.$tabs.pageList.length === 1) {
        this.$message.warning("这是最后一页，不能再关闭了啦");
        return;
      }
      this.$root.$tabs.pageList = this.$root.$tabs.pageList.filter(
        item => item.fullPath !== key
      );
      let index = this.$root.$tabs.linkList.indexOf(key);
      this.$root.$tabs.linkList = this.$root.$tabs.linkList.filter(
        item => item !== key
      );
      index =
        index >= this.$root.$tabs.linkList.length
          ? this.$root.$tabs.linkList.length - 1
          : index;
      this.$root.$tabs.activePage = this.$root.$tabs.linkList[index];
    }
  }
};
</script>

<style lang="scss">
.page-transition-enter {
  opacity: 0;
}
.page-transition-leave-active {
  opacity: 0;
}
.page-transition-enter .page-transition-container,
.page-transition-leave-active .page-transition-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
/*美化弹出Tab样式*/
.ant-tabs-nav-container {
  margin-top: 4px;
}
/* 修改 ant-tabs 样式 */
.tab-layout-tabs.ant-tabs {
  border-bottom: 1px solid #ccc;
  border-left: 1px solid #ccc;
  background-color: white;
  padding: 0 20px;

  .ant-tabs-bar {
    margin: 4px 0 0;
    border: none;
  }
}
.ant-tabs {
  &.ant-tabs-card .ant-tabs-tab {
    padding: 0 24px !important;
    background-color: white !important;
    margin-right: 10px !important;

    .ant-tabs-close-x {
      width: 12px !important;
      height: 12px !important;
      opacity: 0 !important;
      cursor: pointer !important;
      font-size: 12px !important;
      margin: 0 !important;
      position: absolute;
      top: 36%;
      right: 6px;
    }

    &:hover .ant-tabs-close-x {
      opacity: 1 !important;
    }
  }
}
.ant-tabs.ant-tabs-card > .ant-tabs-bar {
  .ant-tabs-tab {
    border: none !important;
    border-bottom: 1px solid transparent !important;
  }
  .ant-tabs-tab-active {
    border-color: #1890ff !important;
  }
}
</style>
