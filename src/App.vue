<template>
  <!-- 顶部面包屑 -->
  <nav-header v-if="!isHideMenu"></nav-header>
  <div>
    <nav-aside v-if="!isHideMenu"></nav-aside>
    <div
      class="moduleContent navCollapse"
      :style="{
        width: `calc( 100% - ${navWidth}px)`,
        top: `${isLogin ? 46 : 0}px`,
      }"
    >
      <router-view />
    </div>
    <div class="moduleContent navCollapse" v-show="!isHideMenu">
      <main-app />
    </div>
  </div>
  <!-- <router-view /> -->
</template>
<script lang="ts">
import { onMounted, reactive, toRefs, watch, defineComponent } from "vue";
// import { onBeforeRouteUpdate } from "vue-router";
import NavHeader from "@/components/NavHeader.vue";
import NavAside from "@/components/NavAside.vue";
import MainApp from "@/components/Main.vue";
import { useStore } from "vuex";
import { useRoute, useRouter, onBeforeRouteUpdate } from "vue-router";
import { GetMenuItems } from "@/utils/authorization";
// definComponent主要是用来帮助Vue在TS下正确推断出setup()组件的参数类型;引入 defineComponent() 以正确推断 setup() 组件的参数类型；defineComponent 可以正确适配无 props、数组 props 等形式；
export default defineComponent({
  components: {
    NavHeader,
    NavAside,
    MainApp,
  },
  setup() {
    const state = reactive({
      isHideMenu: true,
      navWidth: 100,
      isLogin: true,
      erd: "",
    });
    const store = useStore();
    const route = useRoute();
    const router = useRouter();
    watch(route, () => {
      if (Object.prototype.hasOwnProperty.call(route.meta, "isLogin")) {
        console.log("监听router的跳转");
        state.isHideMenu = true;
      } else {
        state.isHideMenu = false;
      }
    });
    // isLogin(){
    //     return this.store.getters.isLogin
    // }
    // watch(
    //   () => store,
    //   (val, old) => {
    //     console.log("watch监听store", store);
    //   }
    // );
    onBeforeRouteUpdate((to) => {
      console.log("=====", to);
    });
    console.log("router.options.routes", router.options.routes);
    onMounted(async () => {
      (window as any).route = (path: string, query: any = {}) => {
        // router.push({ path, query: query });
      };
      // 如果当前路由meta上不存在isLogin
      // console.log("route.meta",route.meta)
      if (!Object.prototype.hasOwnProperty.call(route.meta, "isLogin")) {
        if (sessionStorage.getItem("token")) {
          const menus = await GetMenuItems();
          store.commit("updateMenus", { Menus: menus });
        } else {
          sessionStorage.clear();
          localStorage.clear();
          console.log(
            "如果当前路由meta上不存在isLogin时触发router.push--login"
          );
          // router.push({ path: "/login" });
        }
      }
      // setInterval(() => {
      //   state.erd.listenTo(
      //     document.getElementsByClassName("leftNav"),
      //     (element: any) => {
      //       state.navWidth = element.offsetWidth - 1;
      //     }
      //   );
      // }, 100);
    });
    return {
      ...toRefs(state),
      // getTestInfo,
    };
  },
});
</script>
<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
.moduleContent {
  width: calc(100% - 65px);
  position: fixed;
  right: 0;
  bottom: 0;
  overflow: auto;
  .navCollapse {
    -moz-transition: width 0.3s linear;
    -webkit-transition: width 0.3s linear;
    transition: width 0.3s linear;
  }
}
</style>
