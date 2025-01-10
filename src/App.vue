<template>
  <div class="ld-div">
    <div v-if="isMobile" style="height: 100%;width: 100%;background-color: #000;"></div>
    <LuckDraw v-else />
  </div>
</template>

<script>
import LuckDraw from './components/LuckDraw.vue';
import judgeDevice from '@/tools/judgeDevice';

export default {
  name: 'App',
  components: {
    LuckDraw
  },
  data() {
    return {
      isMobile: false,
    }
  },
  mounted() {
    this.isMobile = judgeDevice();
    if (this.isMobile) {
      alert("当前项目暂未适配移动端，请在pc端打开！")
    }
  },
}
</script>

<style lang="scss">
.ld-div {
  height: 100vh;
  width: 100vw;
}
</style>
