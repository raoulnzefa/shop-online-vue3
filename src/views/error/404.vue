<script lang="ts" setup>
import { useRouter } from 'vue-router';
import IMAGE_404 from '@/assets/images/error/404.png';

const router = useRouter();

function goHome() {
  router.replace('/home');
}
</script>

<template>
  <div class="container">
    <van-empty :image="IMAGE_404" description="对不起，你访问的页面已被删除或不存在。">
      <van-button @click="goHome">去首页</van-button>
    </van-empty>
  </div>
</template>

<style lang="less" scoped>
.container {
  box-sizing: border-box;
  min-height: 100vh;
  background: var(--white);
}
</style>
