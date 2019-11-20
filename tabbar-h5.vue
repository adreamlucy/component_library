<template>
  <div id="app">
    <router-view />
    <div class="tabbar">
      <div :class="active  === index ? 'tabar-item active-item' : 'tabar-item'"
        v-for="(item, index) in tabbar" :key="index" @click="jumpTo(item, index)">
        <div class="item-img" v-if="!item.icon"><img
            :src=" active  === index ? item.active_img : item.default_img" /></div>
        <van-icon class="item-icon iconfont" class-prefix='icon'
          :name=" active  === index ? item.active_img : item.default_img" size="36"
          v-if="item.icon" />
        <div class="item-title" v-if="item.icon">{{ item.title }}</div>
        <div class="item-title item-custom-title" v-if="!item.icon">{{ item.title }}</div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Type from './types/index';
@Component
export default class App extends Vue {
  active: number = 0;
  tabbar: Array<Type.tabbarItem> = [
    {
      title: '寻 狗',
      default_img: 'search',
      active_img: 'search',
      jump: 'find_dog',
      icon: true,
      active: true
    },
    {
      title: '寻 主',
      default_img: 'calculator',
      active_img: 'calculator-fill',
      jump: 'find_master',
      icon: true,
      active: false
    },
    {
      title: '发 布',
      default_img: require('./assets/imgs/publish.png'),
      active_img: require('./assets/imgs/publish.png'),
      jump: 'find_publish',
      icon: false,
      active: false
    },
    {
      title: '领 养',
      default_img: 'order',
      active_img: 'order-fill',
      jump: 'find_mistress',
      icon: true,
      active: false
    },
    {
      title: '我 的',
      default_img: 'office-supplies',
      active_img: 'office-supplies-fill',
      jump: 'mine',
      icon: true,
      active: false
    }
  ];
  protected jumpTo(item: Type.tabbarItem, index: number): void {
    if (this.active !== index) {
      this.active = index;
      this.$router.push({ name: item.jump });
    }
  }
}
</script>
<style lang="scss" scoped>
#app {
  width: 375px;
  height: 100vh;
  color: #2c3e50;
}
.tabbar {
  width: 100%;
  height: 50px;
  position: fixed;
  bottom: 0px;
  left: 0px;
  z-index: 100;
  display: flex;
  justify-content: space-around;
  align-items: center;
  background: #ffffff;
  .tabar-item {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
    height: 100%;
    .item-title {
      font-size: 10px;
      color: #89939f;
      // active-color: #FA7718;
    }
    &.active-item {
      .item-title {
        color: #1989fa;
      }
    }
    .item-img {
      width: 56px;
      height: 56px;
      position: absolute;
      top: -27px;
      img {
        width: 100%;
        height: 100%;
      }
    }
    .item-custom-title {
      position: absolute;
      bottom: 0px;
    }
  }
}
</style>
