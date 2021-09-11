<script setup>
import { ref, onMounted } from "vue";
import clock from "@/assets/clock.png";
import camera from "@/assets/camera.png";
import tv from "@/assets/tv.png";
import stocks from "@/assets/stocks.png";
import mail from "@/assets/mail.png";
import files from "@/assets/files.png";
import appstore from "@/assets/appstore.png";
import weather from "@/assets/weather.png";
import health from "@/assets/health.png";
import home from "@/assets/home.png";
import podcasts from "@/assets/podcasts.png";
import photos from "@/assets/photos.png";
import phone from "@/assets/phone.png";
import safari from "@/assets/safari.png";
import message from "@/assets/message.png";
import music from "@/assets/music.png";
import tiktok from "@/assets/tiktok.png";

const appList = [
  { img: clock, label: "时钟" },
  { img: camera, label: "相机" },
  { img: tv, label: "视频" },
  { img: stocks, label: "股市" },
  { img: mail, label: "邮件" },
  { img: files, label: "文件" },
  { img: appstore, label: "App Store" },
  { img: weather, label: "天气" },
  { img: health, label: "健康" },
  { img: home, label: "家庭" },
  { img: podcasts, label: "播客" },
  { img: photos, label: "照片" },
  { img: tiktok, label: "抖音" },
];
const appListDock = [
  { img: phone },
  { img: safari },
  { img: message },
  { img: music },
];
const textArray = ["想", "你", "了"];
const symbolArray = ["!", "...", "?", "~", "@", "#", "$", "%", "^", "&", "*"];
const fallingOrder = [
  4, 10, 8, 9, 14, 1, 7, 15, 3, 12, 6, 10, 5, 11, 2, 13, 16, 0,
];
const stepList = [
  {
    desc: "时钟",
    time: 2000,
  },
  {
    desc: "窗户",
    time: 2000,
  },
  {
    desc: "房屋",
    time: 1000,
  },
];

const text = ref([]);
const symbolRandomTimes = ref(50);
const symbol = ref("");
const toBeDeleted = ref(false);
const powerOn = ref(false);
const showShutdown = ref(false);
const fadeIn = ref(false);
const fadeOut = ref(false);
const showMail = ref(false);
const showApp = ref(false);
const showTip = ref(true);
const fallingList = ref([]);
const step = ref(0);
const beforeScrollTop = ref(0);
const debounceScroll = ref(() => {});

const initMailContent = () => {
  let timer = setTimeout(() => {
    if (textArray.length > 0) {
      text.value.push(textArray.shift());
      initMailContent();
    } else {
      clearTimeout(timer);
      randomSymbol();
    }
  }, 800);
};

const randomSymbol = () => {
  let timer = setTimeout(() => {
    if (symbolRandomTimes.value > 0) {
      let index = Math.floor(Math.random() * symbolArray.length);
      if (symbolRandomTimes.value === 1) {
        index = Math.floor(Math.random() * 4);
      }
      symbolRandomTimes.value--;
      symbol.value = symbolArray[index];
      randomSymbol();
    } else {
      clearTimeout(timer);
    }
  }, 50);
};

const openApp = () => {
  showApp.value = true;
};

const closeApp = () => {
  showApp.value = false;
  toBeDeleted.value = true;
  deleteApps();
};

const scrollStep = ({ target }) => {
  showTip.value = false;
  let timer;
  const { clientHeight, scrollTop } = target;
  const positive = scrollTop - beforeScrollTop.value >= 0;
  const time = clientHeight + scrollTop;
  const currentTime = stepList
    .slice(0, step.value + 1)
    .map((item) => item.time)
    .reduce((sum, cur) => sum + cur);
  beforeScrollTop.value = scrollTop;
  if (positive) {
    if (step.value < stepList.length - 1) {
      if (clientHeight + scrollTop > currentTime) {
        step.value++;
      }
    }
  } else {
    if (step.value > 0) {
      if (scrollTop < currentTime - scrollTop) {
        step.value--;
      }
    }
  }
  if (step.value === 2) {
    timer = setTimeout(() => {
      closeApp();
    }, 5000);
  } else {
    clearTimeout(timer);
  }
};
let timer;
const deleteApps = () => {
  clearTimeout(timer);
  timer = setTimeout(() => {
    if (fallingOrder.length > 0) {
      fallingList.value.push(fallingOrder.shift());
      deleteApps();
    } else {
      shutdown();
    }
  }, 150);
};

const checkMail = () => {
  showMail.value = true;
  initMailContent();
};

const shutdown = () => {
  showShutdown.value = true;
  setTimeout(() => {
    fadeIn.value = true;
  }, 1000);
  setTimeout(() => {
    fadeOut.value = true;
    powerOn.value = true;
    setTimeout(() => {
      showShutdown.value = false;
    }, 1000);
  }, 3000);
};

const debounce = (fn, delay) => {
  let timer;
  return (...args) => {
    const context = this;
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  };
};

onMounted(() => {
  // 绑定 debounce
  debounceScroll.value = debounce(scrollStep, 10);
});
</script>

<template>
  <div
    class="iphone"
    :class="{
      'app-to-be-deleted': toBeDeleted,
    }"
  >
    <div class="background"></div>
    <div class="hardware"></div>
    <div class="screen">
      <div class="screen__top">
        <img src="@/assets/top_ui.png" alt="top_ui" />
      </div>
      <div class="screen__center">
        <ul class="app-list">
          <li
            v-for="(item, index) of appList"
            :key="index"
            :class="{
              falling: fallingList.indexOf(index) !== -1,
            }"
            @click="openApp"
          >
            <div class="img">
              <img :src="item.img" alt="" />
            </div>
            <span class="label">{{ item.label }}</span>
          </li>
        </ul>
      </div>
      <div class="screen__bottom">
        <div class="dock">
          <ul class="app-list">
            <li
              v-for="(item, index) of appListDock"
              :key="index"
              :class="{
                falling: fallingList.indexOf(index + appList.length) !== -1,
              }"
            >
              <div class="img">
                <img :src="item.img" alt="" />
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
  <div v-if="powerOn" class="phone--green-screen" @click="checkMail">
    <div class="background"></div>
    <div class="screen">
      <div class="mail">
        <div class="mail__content" v-if="showMail">
          <span>讯息：</span>
          <p>
            {{ text.join("") }}<span>{{ symbol }}</span>
          </p>
        </div>
        <img v-else src="@/assets/mail.svg" alt="" />
      </div>
    </div>
  </div>
  <div
    v-if="showShutdown"
    class="phone--shutdown"
    :class="{ 'fade-in': fadeIn, 'fade-out': fadeOut }"
  >
    <div class="icon">
      <img src="@/assets/icon-loading.svg" alt="" />
      <div class="animation"></div>
    </div>
  </div>
  <div class="app" :class="{ 'scale-fade-in': showApp }">
    <div v-if="showTip" class="scroll-tip">
      <img src="@/assets/scroll-finger.png" alt="" />
    </div>
    <div class="scroll" @scroll="debounceScroll">
      <div
        :style="{
          height: `${stepList
            .map((item) => item.time)
            .reduce((sum, cur) => sum + cur)}px`,
        }"
      ></div>
    </div>
    <div
      :class="{
        'scale-fade-out': step !== 0,
      }"
      class="clock"
    >
      <span
        class="clock__hand clock__hand-hour"
        :style="{
          transform: `rotate(${(beforeScrollTop / 6) % 360}deg)`,
        }"
      ></span>
      <span
        class="clock__hand clock__hand-minute"
        :style="{
          transform: `rotate(${beforeScrollTop % 360}deg)`,
        }"
      ></span>
    </div>
    <div
      :class="{
        'window-move-in': step > 0,
      }"
      class="window"
    >
      <img src="@/assets/window.jpg" alt="" />
      <div class="sun-set-rise">
        <div
          :style="{
            backgroundColor:
              beforeScrollTop % 360 < 180 ? 'transparent' : 'black',
          }"
          class="filter"
        ></div>
        <img class="bg" src="@/assets/sun-moon-bg.png" alt="" />
        <img
          :style="{
            transform: `rotate(${beforeScrollTop % 360}deg)`,
          }"
          class="animation"
          src="@/assets/sun-moon.png"
          alt=""
        />
      </div>
      <div
        :class="{
          'fade-in': step > 1,
          'view-scale-full': step > 1,
        }"
        class="view"
      >
        <img src="@/assets/window.jpg" alt="" />
      </div>
    </div>
  </div>
</template>

<style>
img {
  display: block;
}
</style>

<style lang="less" scoped>
.iphone,
.phone--shutdown,
.phone--green-screen,
.app {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
}

.iphone {
  .background,
  .hardware,
  .screen {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-size: 100% 100%;
  }

  .background {
    z-index: -1;
    background-image: url("@/assets/screen-bg.jpg");
  }

  .hardware {
    z-index: 0;
    background-image: url("@/assets/hardware.png");
  }

  .screen {
    z-index: 1;
    margin: 24px;

    .screen__top {
      padding: 8px 12px 0 18px;
      & > img {
        display: block;
        width: 100%;
      }
    }

    .screen__center {
      padding: 30px 10px;
    }

    .screen__bottom {
      position: absolute;
      left: 0;
      right: 0;
      bottom: 0;
      padding: 12px;
      .dock {
        margin: 0 auto;
        padding: 6px;
        background-color: rgba(0, 0, 0, 0.5);
        border-radius: 30px;

        .app-list {
          & > li {
            & > img {
              width: 100%;
            }
          }
        }
      }
    }
  }
}

.phone--shutdown {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #000;
  opacity: 0;
  transition: opacity 1s;

  & > .icon {
    position: relative;
    width: 20px;

    & > img {
      width: 100%;
    }

    & > .animation {
      position: absolute;
      top: 0;
      left: 0;
      width: 20px;
      height: 20px;
      animation: rotate 1s linear infinite;

      &::after {
        content: "";
        display: block;
        width: 10px;
        height: 10px;
        background-color: rgba(0, 0, 0, 0.6);
        border-radius: 50%;
      }
    }
  }
}

.phone--green-screen {
  background-color: #000;
  .background,
  .screen {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-size: 100% 100%;
  }

  & > .background {
    background-image: url("@/assets/nokia.jpg");
  }

  & > .screen {
    padding: 6vh 20px 0 20px;
    opacity: 0.7;

    .mail {
      text-align: center;

      & > img {
        margin: 5vh auto 0 auto;
      }

      .mail__content {
        margin: 15vh 20px;
        text-align: left;
        font-size: 28px;
      }
    }
  }
}

.app {
  opacity: 0;
  background-color: #fff;
  transform: scale(0);
  transition: all 0.3s;

  .scroll-tip {
    position: absolute;
    top: 40%;
    right: 10px;
    z-index: 10;
    animation: upAndDown 2s ease-out infinite;
    & > img {
      width: 20vw;
    }
  }

  .clock,
  .window,
  .scroll {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }

  .scroll {
    z-index: 999;
    overflow-y: scroll;
  }

  .clock,
  .window {
    background-repeat: no-repeat;
    background-position: center;
    background-size: 100% auto;
  }

  .clock {
    z-index: 0;
    background-image: url("@/assets/clock2.png");
    transform-origin: 85vw 40vh;
    transition: all 2s;

    .clock__hand {
      position: absolute;
      top: 50%;
      left: 50%;
      display: block;
      background-color: #000;
      transform-origin: 4px 4px;
    }

    .clock__hand-hour {
      width: 60px;
      height: 8px;
      border-radius: 8px;
    }

    .clock__hand-minute {
      width: 90px;
      height: 5px;
      border-radius: 5px;
    }
  }

  .window {
    top: 50%;
    z-index: 1;
    transform-origin: 90vw 13vh;
    transform: translateY(-50%) scale(6.5);
    opacity: 0;
    transition: all 2s;

    & > img {
      width: 100%;
      transition: all 3s;
    }

    .sun-set-rise {
      position: absolute;
      top: 0;
      overflow: hidden;
      .bg {
        position: relative;
        z-index: 2;
        width: 100%;
      }

      .filter {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 0;
        opacity: 0.5;
      }

      .animation {
        position: absolute;
        top: 80px;
        left: 28px;
        z-index: 1;
      }
    }

    .view {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 2;
      display: flex;
      align-items: center;
      transform-origin: 30% 50%;
      opacity: 0;
      transition: transform 5s;

      & > img {
        width: 100%;
      }
    }

    .view-scale-full {
      transform: scale(10);
    }
  }
}

.app-to-be-deleted {
  .app-list {
    & > li {
      &::after {
        content: "x";
        position: absolute;
        top: -8px;
        left: -8px;
        display: block;
        width: 20px;
        height: 20px;
        background-color: #888;
        border-radius: 50%;
        text-align: center;
        line-height: 18px;
        font-size: 14px;
        color: #000;
      }
      & > .img {
        animation: shake 0.5s linear infinite;
      }
    }
  }
}

.app-list {
  display: flex;
  flex-wrap: wrap;
  margin: 0;
  padding: 0;
  list-style: none;

  & > li {
    position: relative;
    width: calc(25% - 20px);
    margin: 8px 10px;

    & > .img {
      position: relative;

      & > img {
        display: block;
        width: 90%;
        margin: 0 auto;
      }
    }

    & > .label {
      display: block;
      margin-top: 6px;
      text-align: center;
      font-size: 12px;
      color: #fff;
    }
  }
}

.window-move-in {
  transform: translateY(-50%) scale(1) !important;
  opacity: 1 !important;
}

.scale {
  transform: scale(1) !important;
}

.scale-fade-in {
  opacity: 1 !important;
  transform: scale(1) !important;
}

.scale-fade-out {
  opacity: 0 !important;
  transform: scale(0) !important;
}

.falling {
  animation: fall 2s linear;
  animation-fill-mode: forwards;
}

.fade-in {
  opacity: 1 !important;
}

.fade-out {
  opacity: 0 !important;
}

@keyframes shake {
  25% {
    transform: rotate(8deg);
  }
  50% {
    transform: rotate(0deg);
  }
  75% {
    transform: rotate(-8deg);
  }
  100% {
    transform: rotate(0deg);
  }
}

@keyframes fall {
  100% {
    transform: translateY(1000px);
  }
}

@keyframes rotate {
  100% {
    transform: rotate(360deg);
  }
}

@keyframes upAndDown {
  from {
    transform: translateY(20vh);
  }
  to {
    transform: translateY(0);
  }
}
</style>
