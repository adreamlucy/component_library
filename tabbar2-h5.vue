<template>
  <div id="app">
    <router-view />
    <div class="tabbar">
      <div :class="active === index ? 'tabar-item active-item' : 'tabar-item'"
        v-for="(item, index) in tabbar" :key="index" @click="jumpTo(item, index)">
        <div class="item-title" v-if="!item.img">{{item.title}}</div>
        <div class="item-line" v-if="!item.img"></div>
        <div v-if="item.img" class="item-img">
          <img :src="item.img" alt="">
        </div>
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
      title: '寻狗',
      jump: 'find_dog'
    },
    {
      title: '寻主',
      jump: 'find_master'
    },
    {
      title: '发布',
      img: require('./assets/imgs/publish.png'),
      jump: 'find_publish'
    },
    {
      title: '领养',
      jump: 'find_mistress'
    },
    {
      title: '我的',
      jump: 'mine'
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
    // position: relative;
    height: 100%;
    .item-title {
      font-size: 20px;
    }
    .item-line {
      width: 20px;
      // background: #159df9;
      height: 4px;
      border-radius: 4px;
    }
    .item-img {
      width: 56px;
      height: 56px;
      margin-bottom: 33px;
      img {
        width: 100%;
        height: 100%;
      }
    }
    &.active-item {
      .item-title {
        font-size: 20px;
        color: #159df9;
      }
      .item-line {
        background: #159df9;
      }
    }
  }
}
</style>
