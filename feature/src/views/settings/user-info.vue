<template>
  <div class="user-info">
<!--    <div class="info-container">-->
<!--      <a-result-->
<!--        class="user-info-result"-->
<!--        :title="userInfo.name || $t('feature.settings.account.tips1')"-->
<!--        :sub-title="$t('feature.settings.account.tips2')"-->
<!--      >-->
<!--        <template #icon>-->
<!--          <a-avatar :size="64" v-if="!userInfo.avatar">-->
<!--            <template #icon><UserOutlined /></template>-->
<!--          </a-avatar>-->
<!--          <a-avatar :src="userInfo.avatar" :size="64" v-else />-->
<!--        </template>-->
<!--      </a-result>-->
<!--    </div>-->
    <div class="settings-container">
      <div class="setting-item">
        <div class="title">
          {{ $t('feature.settings.account.themeColor') }}
        </div>
        <div class="settings-item-li">
          <div class="label">
            {{ $t('feature.settings.account.primaryColor') }}
          </div>
          <a-input v-model:value="custom.primaryColor" class="value">
            <template #prefix>
              <div
                :style="{
                  background: custom.primaryColor,
                  width: '10px',
                  height: '10px',
                }"
              ></div>
            </template>
          </a-input>
        </div>
        <div class="settings-item-li">
          <div class="label">
            {{ $t('feature.settings.account.errorColor') }}
          </div>
          <a-input v-model:value="custom.errorColor" class="value">
            <template #prefix>
              <div
                :style="{
                  background: custom.errorColor,
                  width: '10px',
                  height: '10px',
                }"
              ></div>
            </template>
          </a-input>
        </div>
        <div class="settings-item-li">
          <div class="label">
            {{ $t('feature.settings.account.warningColor') }}
          </div>
          <a-input v-model:value="custom.warningColor" class="value">
            <template #prefix>
              <div
                :style="{
                  background: custom.warningColor,
                  width: '10px',
                  height: '10px',
                }"
              ></div>
            </template>
          </a-input>
        </div>
        <div class="settings-item-li">
          <div class="label">
            {{ $t('feature.settings.account.successColor') }}
          </div>
          <a-input v-model:value="custom.successColor" class="value">
            <template #prefix>
              <div
                :style="{
                  background: custom.successColor,
                  width: '10px',
                  height: '10px',
                }"
              ></div>
            </template>
          </a-input>
        </div>
        <div class="settings-item-li">
          <div class="label">
            {{ $t('feature.settings.account.infoColor') }}
          </div>
          <a-input v-model:value="custom.infoColor" class="value">
            <template #prefix>
              <div
                :style="{
                  background: custom.infoColor,
                  width: '10px',
                  height: '10px',
                }"
              ></div>
            </template>
          </a-input>
        </div>
      </div>
      <div class="setting-item">
        <div class="title">
          {{ $t('feature.settings.account.personalized') }}
        </div>
        <div class="settings-item-li">
          <div class="label">
            {{ $t('feature.settings.account.greeting') }}
          </div>
          <a-input v-model:value="custom.placeholder" class="value"></a-input>
        </div>
        <div class="settings-item-li">
          <div class="label">
            {{ $t('feature.settings.account.logo') }}
          </div>
          <div class="img-container">
            <img class="custom-img" :src="custom.logo" />
            <a-button @click="changeLogo" size="small" type="link">
              {{ $t('feature.settings.account.replace') }}
            </a-button>
          </div>
        </div>
      </div>
<!--      <div class="footer-btn">-->
<!--        <a-button @click="reset" type="danger">-->
<!--          {{ $t('feature.settings.account.reset') }}-->
<!--        </a-button>-->
<!--      </div>-->
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, toRefs, watch } from 'vue';
import debounce from 'lodash.debounce';
import localConfig from '@/confOp';

import service from '../../assets/service';

const { ipcRenderer } = window.require('electron');

const state = reactive({
  custom: {},
});

const { perf } = localConfig.getConfig();

state.custom = perf.custom || {};

const userInfo = ref(window.rubick.dbStorage.getItem('rubick-user-info'));

// service.getUserInfo({ openId: userInfo.value.openId }).then((res) => {
//   userInfo.value = res;
// });

const setConfig = debounce(() => {
  localConfig.setConfig(
    JSON.parse(
      JSON.stringify({
        perf: {
          ...perf,
          custom: state.custom,
        },
      })
    )
  );
  ipcRenderer.send('re-register');
}, 500);

watch(state, setConfig);
const { custom } = toRefs(state);

const changeLogo = () => {
  const [logoPath] = window.rubick.showOpenDialog({
    title: '请选择 logo 路径',
    filters: [{ name: 'images', extensions: ['png'] }],
    properties: ['openFile'],
  });
  state.custom.logo = `file://${logoPath}`;
};

// const reset = () => {
//   Modal.warning({
//     title: '确定恢复默认设置吗？',
//     content: '回复后之前的设置将会被清空',
//     onOk() {
//       const defaultcustom = remote.getGlobal('OP_CONFIG').getDefaultConfig()
//         .perf.custom;
//       state.custom = JSON.parse(JSON.stringify(defaultcustom));
//     },
//   });
// };
</script>

<style lang="less">
.settings-container {
  margin-top: 18px;
}
.user-info-result {
  padding: 0;
  &.ant-result {
    padding: 24px;
  }
  .icon {
    font-size: 48px;
  }
  .ant-result-icon {
    margin-bottom: 12px;
  }
  .ant-result-title {
    font-size: 18px;
  }
}
.img-container {
  width: 300px;
}
.custom-img {
  width: 60px;
  height: 60px;
}
.footer-btn {
  text-align: right;
  border-top: 1px dashed #ddd;
  padding-top: 12px;
}
</style>
