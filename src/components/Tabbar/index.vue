<script lang="ts">
import { computed, defineComponent, ref, unref, watchEffect } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useAppStore } from '@/store/modules/app';

export default defineComponent({
  name: 'Tabbar',
  setup() {
    const router = useRouter();
    const route = useRoute();
    const appStore = useAppStore();

    const active = ref('/home');
    const tabBar = computed(() => appStore.getTabBar);
    const tabList = computed(() => unref(tabBar).list);

    watchEffect(() => {
      active.value = route.path;
    });

    function onTabClicked(index) {
      const { pagePath } = unref(tabList)[index];

      router.replace({
        path: pagePath,
      });
    }

    return {
      active,
      tabList,
      onTabClicked,
    };
  },
});
</script>

<template>
  <div class="tabbar-wrap">
    <van-tabbar v-model="active" class="tabbar" fixed>
      <van-tabbar-item v-for="(item, index) in tabList" :key="index" :name="item.pagePath" @click="onTabClicked(index)">
        <template #icon>
          <van-icon :name="item.icon" />
        </template>
        {{ item.text }}
      </van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<style lang="less" scoped>
.tabbar-wrap {
  height: calc(50px + constant(safe-area-inset-bottom));
  height: calc(50px + env(safe-area-inset-bottom));
}

.tabbar {
  :deep(.van-tabbar-item--active) {
    color: var(--brand-color);
    color: var(--brand-color);
  }
}
</style>
