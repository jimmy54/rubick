<template>
  <div class="worker">
    <PluginList
      v-if="system && !!system.length"
      @downloadSuccess="downloadSuccess"
      :title="$t('feature.market.efficiency')"
      :list="system"
    />
  </div>
</template>

<script setup>
import { ref, computed, onBeforeMount } from 'vue';
import request from '../../../assets/request/index';
import PluginList from './plugin-list.vue';

import { useStore } from 'vuex';
const store = useStore();
const totalPlugins = computed(() => store.state.totalPlugins);

const data = ref([]);

onBeforeMount(async () => {
  data.value = await request.getWorkerDetail();
});

const system = computed(() => {
  const defaultData = data.value || [];
  if (!defaultData.length) return [];
  return defaultData.map((plugin) => {
    let searchInfo = null;
    totalPlugins.value.forEach((t) => {
      if (t.name === plugin) {
        searchInfo = t;
      }
    });
    return searchInfo;
  });
});
</script>

<style lang="less">
.worker {
  width: 100%;
  height: 100vh;
  overflow-x: hidden;
  box-sizing: border-box;
}
</style>
