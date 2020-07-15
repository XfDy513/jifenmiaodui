<template>
  <div id="app">
    <div id="header" :style="{height: h,}"></div>
    <div class="viewBox" :style="{top: h}">
      <router-view v-if="isRouterAlive" />
    </div>
    <div v-transfer-dom>
      <x-dialog v-model="activity" id="showActivity" hide-on-blur>
        <div class="showAction">
          <img :src="active.cover" alt class="bg" @click="gourl()" />
          <img src="./assets/img/close.png" class="close" @click="activity = false" />
        </div>
      </x-dialog>
    </div>
  </div>
</template>
<script>
import * as utils from './utils'
import { XDialog, TransferDomDirective as TransferDom } from "vux";
export default {
    components: {
    XDialog
  },
  directives: {
    TransferDom
  },
  name: "App",
  provide() {
    return {
      reload: this.reload
    };
  },
  data() {
    return {
      isRouterAlive: true,
      h: "",
      anh: 25,
      iosh: 20,
      hideTitle: ["home", "my", "sign"],
      version: "",
      transitionName: "",
      active: "",
      activity: false
    };
  },
  watch: {
    $route(to, from) {
      if (this.hideTitle.indexOf(to.name) > -1 || !window.api) {
        this.h = 0;
      } else {
        if (api.systemType === "android") {
          this.h = this.anh;
        } else {
          this.h = this.iosh;
        }
      }
      var bdTTS = api.require("bdTTS");
      bdTTS.speakPause(function(ret) {});
      if (to.name == "home") {
        let text = "欢迎使用积分妙兑智能终端，请选择您要兑换的银行。";
        var bdTTS = api.require("bdTTS");
        bdTTS.speak(
          {
            text: text
          },
          function(ret) {}
        );
      }
      let systemVersion = (window.api && api.systemVersion) || "6.0";
      if (parseFloat(systemVersion) < 6.0) {
        this.colors = "#e65013";
      } else {
        this.colors = "#fff";
      }
    }
  },
  created() {
    let that = this;
    // 进入前台
    let size = document.documentElement.clientWidth / 7.5;
    if (window.api) {
      if (api.systemType === "android") {
        this.h = api.safeArea.top / size + "rem";
        this.anh = this.h;
      } else {
        this.h = api.safeArea.top / size + "rem";
        this.iosh = this.h;
      }
    }
    if (this.hideTitle.indexOf(that.$route.name) > -1) {
      this.h = 0;
    }
    window.api &&
      api.setStatusBarStyle({
        style: "white",
        color: "#e65013"
      });
    let systemVersion = (window.api && api.systemVersion) || "6.0";
    if (parseFloat(systemVersion) < 6.0) {
      that.colors = "#e65013";
    } else {
      that.colors = "#fff";
    }
    that.getAdvertisement();
  },
  methods: {
    reload() {
      this.isRouterAlive = false;
      this.$nextTick(() => {
        this.isRouterAlive = true;
      });
    },
    getAdvertisement() {
      console.log(this.$store.state.global.token)
      if (this.$store.state.global.token) {
        this.$http
          .post("news/getAdvertisement", {token: this.$store.state.global.token})
          .then(res => {
            if (res.code === 1) {
              this.activity = true;
              this.active = res.data;
            }
          });
      }
    },
    gourl(){
      if(this.active.url){
        var browser = api.require("webBrowser");
        this.activity = false
        browser.open({
          url: this.active.url
        });
      }else{
        this.activity = false
      }
    }
  },
  mounted() {
    var bdTTS = api.require("bdTTS");
    bdTTS.init(
      {
        mode: "MIX"
      },
      function(ret) {}
    );
    var bdTTS = api.require("bdTTS");
    bdTTS.speakPause(function(ret) {});
    let text = "欢迎使用积分妙兑智能终端，请选择您要兑换的银行。";
    var bdTTS = api.require("bdTTS");
    bdTTS.speak(
      {
        text: text
      },
      function(ret) {}
    );
  }
};
</script>

<style lang="less">
#showActivity .weui-dialog {
  background-color: transparent;
  z-index: 5010;
}
.showAction {
  .close{
    margin-top: 0.2rem;
    width: 0.5rem;
  }
}
</style>

<style lang="less">
@import "~vux/src/styles/reset.less";
@import "~vux/src/styles/1px.less";
@import "./assets/less/app.less";
@import "./assets/less/font.css";

#app {
  height: 100%;
  position: relative;
  #header {
    position: relative;
    z-index: 1000000;
    overflow: hidden;
    width: 100%;
    /*height: 25px;*/
  }
  .viewBox {
    position: absolute;
    left: 0;
    top: 25px;
    width: 100%;
    bottom: 0;
  }
}

/*上拉加载无数据的时候图片的样式*/
.mescroll-empty {
  padding-top: 130px !important;
}
.mescroll-empty .empty-icon {
  width: 35% !important;
}

//  进入页面路由动画定义
@keyframes slideInLeft {
  from {
    transform: translate3d(100%, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1;
  }
  to {
    transform: translate3d(0, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1;
  }
}
@keyframes slideInRight {
  from {
    transform: translate3d(0%, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1;
  }
  to {
    transform: translate3d(-100%, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1;
  }
}
.slide-left-enter-active {
  position: fixed;
  top: 0;
  left: 0%;
  width: 100vw;
  height: 100vh;
  animation: slideInLeft 0.2s linear forwards;
}
.slide-left-leave-active {
  position: fixed;
  top: 0;
  left: 0%;
  width: 100vw;
  height: 100vh;
  animation: slideInRight 0.2s linear forwards;
}
@keyframes slideOutLeft {
  from {
    transform: translate3d(-100%, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1 !important;
  }
  to {
    transform: translate3d(0%, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1;
  }
}
@keyframes slideOutRight {
  from {
    transform: translate3d(0%, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1;
  }
  to {
    transform: translate3d(100%, 0, 0);
    position: fixed;
    top: 0;
    left: 0;
    opacity: 1;
  }
}
.slide-right-enter-active {
  position: fixed;
  top: 0;
  left: 0%;
  width: 100vw;
  height: 100vh;
  animation: slideOutLeft 0.2s linear forwards;
}
.slide-right-leave-active {
  position: fixed;
  top: 0;
  left: 0%;
  width: 100vw;
  height: 100vh;
  animation: slideOutRight 0.2s linear forwards;
}
</style>