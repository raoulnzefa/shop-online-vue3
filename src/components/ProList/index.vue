<script lang="ts">
import { defineComponent, onMounted, PropType, ref, unref } from 'vue';
import { isFunction } from '@/utils/validate';

export default defineComponent({
  name: 'ProList',
  props: {
    // 接口请求Promise对象
    api: {
      type: Function as PropType<(...arg: any[]) => Promise<any>>,
      required: true,
      default: null,
    },
    // 自定义处理接口返回数据
    afterFetch: {
      type: Function as PropType<Fn>,
      default: null,
    },
    // 分页配置
    pagination: {
      type: Object,
      default: () => ({
        pageCurrent: 1,
        pageSize: 20,
      }),
    },
    // 立即请求接口
    immediate: {
      type: Boolean,
      default: true,
    },
    emptyImage: {
      type: String,
      default: '',
    },
    emptyText: {
      type: String,
      default: '暂无数据',
    },
    errorText: {
      type: String,
      default: '请求失败，点击重新加载',
    },
    loadingText: {
      type: String,
      default: '加载中...',
    },
    finishedText: {
      type: String,
      default: '没有更多了',
    },
    offset: {
      type: [Number, String],
      default: 300,
    },
    direction: {
      type: String,
      default: 'down',
    },
  },
  setup(props, { expose }) {
    const list = ref<Recordable[]>([]);
    const loading = ref(false);
    const error = ref(false);
    const finished = ref(false);

    function deleteItemByIndex(index: number) {
      list.value.splice(index, 1);
    }

    function refresh() {
      if (unref(loading)) {
        return;
      }

      // eslint-disable-next-line vue/no-mutating-props
      props.pagination.pageCurrent = 1;
      list.value = [];
      finished.value = false;
      onLoad();
    }

    function onLoadMore() {
      if (unref(finished)) {
        return;
      }
      // eslint-disable-next-line vue/no-mutating-props
      props.pagination.pageCurrent += 1;
      onLoad();
    }

    function onLoad() {
      loading.value = true;

      props
        .api()
        .then((res) => {
          let records = res.data?.result ?? [];
          const total = res.data?.totalRow ?? 0;

          if (props.afterFetch && isFunction(props.afterFetch)) {
            records = props.afterFetch(res.data);
          }

          list.value = props.pagination.pageCurrent === 1 ? records : list.value.concat(records);
          finished.value = list.value.length >= total;
        })
        .catch((err) => {
          console.error(err);
          error.value = true;
        })
        .finally(() => {
          loading.value = false;
        });
    }

    expose({ deleteItemByIndex, refresh, onLoadMore });

    onMounted(() => {
      props.immediate && onLoad(); // 立刻执行
    });

    return {
      list,
      loading,
      error,
      finished,
      // methods
      onLoadMore,
      onLoad,
    };
  },
});
</script>

<template>
  <van-list
    v-model:loading="loading"
    v-model:error="error"
    class="list"
    :finished="finished"
    :finished-text="finishedText"
    :error-text="errorText"
    :immediate-check="false"
    :offset="offset"
    :direction="direction"
    @load="onLoadMore"
  >
    <template v-for="(item, index) in list">
      <slot name="item" v-bind="{ item, index }"></slot>
    </template>
    <template #finished>
      <span v-if="list.length">{{ finishedText }}</span>
      <van-empty v-else :image="emptyImage || 'default'" :description="emptyText" />
    </template>
  </van-list>
</template>
