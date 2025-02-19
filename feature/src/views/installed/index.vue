<template>
  <div class="installed">
    <div v-if="!localPlugins.length">
      <a-result
        status="404"
        :title="$t('feature.installed.tips1')"
        :sub-title="$t('feature.installed.tips2')"
      />
    </div>
    <div class="container" v-else>
      <div class="installed-list">
        <div
          :class="currentSelect[0] === index ? 'item active' : 'item'"
          :key="index"
          @click="currentSelect = [index]"
          v-for="(plugin, index) in localPlugins"
        >
          <img :src="plugin.logo" />
          <div class="info">
            <div class="title">
              {{ plugin.pluginName }}
              <span class="desc">v{{ plugin.version }}</span>
            </div>
            <div class="desc">{{ plugin.description }}</div>
          </div>
        </div>
      </div>
      <div class="plugin-detail">
        <div class="plugin-top">
          <div class="left">
            <div class="title">
              {{ pluginDetail.pluginName }}
              <a-tag>{{ pluginDetail.version }}</a-tag>
            </div>
            <div class="desc">
              {{ $t('feature.installed.developer') }}：{{
                `${pluginDetail.author || $t('feature.installed.unknown')}`
              }}
            </div>
            <div class="desc">
              {{ pluginDetail.description }}
            </div>
          </div>
          <div class="right">
            <a-button
              type="danger"
              size="small"
              shape="round"
              :loading="pluginDetail.isloading"
              @click="deletePlugin(pluginDetail)"
            >
              {{ $t('feature.installed.remove') }}
            </a-button>
          </div>
        </div>
        <a-tabs default-active-key="1">
          <a-tab-pane key="1" :tab="$t('feature.installed.functionKey')">
            <div class="feature-container">
              <div
                class="desc-item"
                :key="index"
                v-for="(item, index) in pluginDetail.features"
              >
                <div>{{ item.explain }}</div>
                <a-tag
                  :key="cmd"
                  v-for="cmd in item.cmds"
                  :class="{ executable: !cmd.label }"
                >
                  <span
                    @click="
                      !cmd.label &&
                        openPlugin({
                          code: item.code,
                          cmd,
                        })
                    "
                  >
                    {{ cmd.label || cmd }}
                  </span>
                  <template v-if="!cmd.label" #icon>
                    <a-tooltip
                      v-if="!hasAdded(cmd)"
                      placement="topLeft"
                      :title="$t('feature.installed.addToPanel')"
                    >
                      <PlusCircleOutlined
                        @click="addCmdToSuperPanel({ code: item.code, cmd })"
                      />
                    </a-tooltip>
                    <a-tooltip
                      v-else
                      placement="topLeft"
                      :title="$t('feature.installed.removeFromPanel')"
                    >
                      <MinusCircleOutlined
                        @click="removePluginToSuperPanel({ cmd })"
                      />
                    </a-tooltip>
                  </template>
                </a-tag>
              </div>
            </div>
          </a-tab-pane>
          <a-tab-pane key="2" :tab="$t('feature.installed.detailInfo')">
            <div class="detail-container" v-html="readme"></div>
          </a-tab-pane>
        </a-tabs>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useStore } from 'vuex';
import { computed, ref, toRaw } from 'vue';
import path from 'path';
import MarkdownIt from 'markdown-it';
import { PlusCircleOutlined, MinusCircleOutlined } from '@ant-design/icons-vue';
import { message } from 'ant-design-vue';

const { ipcRenderer } = window.require('electron');

const remote = window.require('@electron/remote');
const fs = window.require('fs');
const md = new MarkdownIt();

const appPath = remote.app.getPath('userData');
const baseDir = path.join(appPath, './rubick-plugins');

const store = useStore();
const localPlugins = computed(() =>
  store.state.localPlugins.filter(
    (plugin) => plugin.name !== 'rubick-system-feature'
  )
);
const updateLocalPlugin = () => store.dispatch('updateLocalPlugin');
const startUnDownload = (name) => store.dispatch('startUnDownload', name);
const errorUnDownload = (name) => store.dispatch('errorUnDownload', name);

const currentSelect = ref([0]);

const pluginDetail = computed(() => {
  return localPlugins.value[currentSelect.value] || {};
});

const superPanelPlugins = ref(
  window.rubick.db.get('super-panel-user-plugins') || {
    data: [],
    _id: 'super-panel-user-plugins',
  }
);

const addCmdToSuperPanel = ({ cmd, code }) => {
  const plugin = {
    ...toRaw(pluginDetail.value),
    cmd,
    ext: {
      code,
      type: 'text',
      payload: null,
    },
  };
  superPanelPlugins.value.data.push(plugin);
  window.rubick.db.put(toRaw(superPanelPlugins.value));
};

const removePluginToSuperPanel = ({ cmd, name }) => {
  superPanelPlugins.value.data = toRaw(superPanelPlugins.value).data.filter(
    (item) => {
      if (name) return item.name !== name;
      return item.cmd !== cmd;
    }
  );
  window.rubick.db.put(toRaw(superPanelPlugins.value));
};

const hasAdded = (cmd) => {
  let added = false;
  superPanelPlugins.value.data.some((item) => {
    if (item.cmd === cmd) {
      added = true;
      return true;
    }
    return false;
  });
  return added;
};

const openPlugin = ({ cmd, code }) => {
  window.rubick.openPlugin(
    JSON.parse(
      JSON.stringify({
        ...pluginDetail.value,
        cmd,
        ext: {
          code,
          type: 'text',
          payload: null,
        },
      })
    )
  );
};

const readme = computed(() => {
  if (!pluginDetail.value.name) return '';
  const readmePath = path.resolve(
    baseDir,
    'node_modules',
    pluginDetail.value.name,
    'README.md'
  );
  if (fs.existsSync(readmePath)) {
    const str = fs.readFileSync(readmePath, 'utf-8');
    return md.render(str);
  }
  return '';
});

const deletePlugin = async (plugin) => {
  startUnDownload(plugin.name);
  const timer = setTimeout(() => {
    errorUnDownload(plugin.name);
    message.error('卸载超时，请重试！');
  }, 20000);
  await window.market.deletePlugin(plugin);
  removePluginToSuperPanel({ name: plugin.name });
  updateLocalPlugin();
  clearTimeout(timer);
};


</script>

<style lang="less" scoped>
.installed {
  box-sizing: border-box;
  width: 100%;
  overflow: hidden;
  background: var(--color-body-bg);
  height: calc(~'100vh - 46px');
  :deep(.ant-result-title) {
    color: var(--color-text-primary);
  }
  :deep(.ant-result-subtitle) {
    color: var(--color-text-desc);
  }

  .container {
    box-sizing: border-box;
    width: 100%;
    overflow: hidden;
    background: #f3efef;
    height: 100%;
    display: flex;
  }

  .installed-list {
    width: 40%;
    background: var(--color-body-bg);
    height: 100%;
    padding: 10px 0;
    border-right: 1px solid var(--color-border-light);
    overflow: auto;

    .item {
      padding: 10px 20px;
      display: flex;
      align-items: center;
      color: var(--color-text-content);
      img {
        width: 40px;
        height: 40px;
        margin-right: 20px;
      }

      .desc {
        color: var(--color-text-desc);
      }

      &.active {
        background: var(--color-list-hover);
      }
    }
  }

  .plugin-detail {
    padding: 20px 20px 0 20px;
    box-sizing: border-box;
    width: 60%;
    height: 100%;
    background: var(--color-body-bg);
    .plugin-top {
      display: flex;
      align-items: flex-start;
      justify-content: space-between;

      .title {
        font-size: 20px;
        display: flex;
        align-items: center;
        color: var(--color-text-primary);
        .ant-tag {
          background: var(--color-input-hover);
          border: 1px solid var(--color-border-light);
          color: var(--color-text-content);
          margin-left: 8px;
        }
      }

      .desc {
        font-size: 13px;
        color: var(--color-text-desc);
      }
    }
    .ant-tabs {
      :deep(.ant-tabs-bar) {
        color: var(--color-text-content);
        border-bottom: 1px solid var(--color-border-light);
      }
    }

    .detail-container,
    .feature-container {
      height: 380px;
      overflow: auto;
      color: var(--color-text-content);
      img {
        width: 100%;
      }
    }

    .desc-item {
      border-bottom: 1px solid var(--color-border-light);
      padding: 10px 0;
      color: var(--color-text-content);
      .ant-tag {
        margin-top: 6px;

        &.executable {
          cursor: pointer;
          color: var(--ant-info-color);
          &:hover {
            transform: translateY(-2px);
          }
        }
      }

      .desc-title {
        display: flex;
        align-items: center;
        justify-content: space-between;
      }

      .desc-info {
        color: var(--color-text-desc);
      }
    }
  }
}
</style>
